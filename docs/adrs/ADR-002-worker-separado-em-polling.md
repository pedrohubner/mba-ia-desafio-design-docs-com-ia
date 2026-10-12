# ADR-002: Worker em processo separado, consumindo a outbox por polling de 2 segundos

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-001](ADR-001-outbox-transacional-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md)

## Status

Aceito. Larissa registrou o polling de 2s em [09:10] e o processo separado foi acordado em [09:11]. Ambos foram confirmados no resumo de [09:48].

## Contexto

Com a outbox definida no [ADR-001](ADR-001-outbox-transacional-no-mysql.md), falta decidir **como** e **onde** os eventos são consumidos. O requisito de produto é latência abaixo de 10 segundos ([09:02] Marcos).

O MySQL não tem listener nativo equivalente ao `LISTEN/NOTIFY` do PostgreSQL. Um trigger de banco executa SQL, mas não avisa um processo externo ([09:09] Diego).

A API hoje sobe como um único processo Node a partir de `src/server.ts`. Se o consumidor rodar dentro desse processo, todo restart da API derruba o consumo ([09:11] Diego).

## Direcionadores da Decisão

- Cumprir a latência abaixo de 10s com margem ([09:02] Marcos).
- Não depender de recurso que o MySQL não oferece ([09:09] Diego).
- Ciclo de vida do consumidor independente do ciclo de vida da API ([09:11] Diego).
- Mesma stack e mesmo banco, sem infraestrutura nova ([09:11] Diego, [09:11] Bruno).

## Alternativas Consideradas

1. **Trigger no banco para reagir à inserção** ([09:09] Bruno).
2. **Consumidor dentro do processo da API** (mesma instância de `src/server.ts`).
3. **Processo worker dedicado, com polling de 2s** ([09:09] Diego, [09:11] Larissa).

## Decisão

Adotamos a **alternativa 3**:

- Um novo entry-point `src/worker.ts` (novo), no mesmo molde de `src/server.ts`, com script `npm run worker` ([09:11] Larissa). A lógica de processamento fica dentro do módulo, em `src/modules/webhooks/webhook.worker.ts` ou `webhook.processor.ts` (novos) ([09:28] Bruno).
- O worker faz polling **a cada 2 segundos**, busca os eventos pendentes mais antigos em lote pequeno, processa e marca ([09:09] Diego, [09:08] Diego).
- Mesmo banco e mesma `DATABASE_URL`, mas com **uma instância própria de `PrismaClient`**, porque o PrismaClient é por processo ([09:30] Bruno).
- **Um único worker** em execução. Assim a ordem de entrega segue o `created_at` da outbox e fica implícita por `order_id` ([09:12] Diego).

## Prós e Contras das Alternativas

### Trigger no banco
- ✅ Reação imediata à inserção.
- ❌ O trigger não notifica processo externo e só executa SQL ([09:09] Diego).
- ❌ Exigiria improviso, como escrever em arquivo ou chamar um endpoint ("fica esquisito", [09:09] Diego).

### Consumidor dentro da API
- ✅ Nenhum processo novo para operar.
- ❌ Restart ou deploy da API derruba o consumo ([09:11] Diego).
- ❌ Disputa CPU e event loop com o atendimento HTTP.

### Worker dedicado em polling
- ✅ Simples, previsível e compatível com o MySQL.
- ✅ 2s de intervalo atende a meta de 10s com folga ([09:09] Diego).
- ❌ Latência mínima de até 2s, aceita explicitamente ([09:10] Larissa).
- ❌ Um processo a mais para implantar e monitorar.

## Consequências

**Positivas:** o consumo sobrevive a restarts da API e pode ser implantado e reiniciado de forma independente. Não entra nenhuma tecnologia nova. O pior caso de detecção (2s) cabe com folga na meta de 10s.

**Negativas / limitações conhecidas:** a ordem vale **só por `order_id` e só enquanto houver um único worker** ([09:13] Larissa). Mesmo assim, sob retry, um evento em backoff pode ser ultrapassado por um mais novo do mesmo pedido (RFC-L-04). Não existe garantia de ordem global, e os clientes não a pediram ([09:14] Marcos). Escalar para vários workers em paralelo quebra essa garantia. Particionar por `order_id` ou usar lock pessimista foi deixado como "problema do futuro" ([09:13] Diego). Polling contínuo gera carga de leitura constante no MySQL, mitigada pelos índices do [ADR-001](ADR-001-outbox-transacional-no-mysql.md).

**Trade-off explícito:** trocamos reatividade (2s de latência mínima) e escalabilidade horizontal imediata por simplicidade operacional e ordem por pedido.

## Referências

- `src/server.ts` (modelo de entry-point e graceful shutdown)
- `src/config/database.ts` (`createPrismaClient`, reaproveitado para a instância do worker)
- `package.json` (scripts `dev`/`start`; o novo `worker` segue o mesmo padrão)
- Transcrição: [09:08]–[09:13] Diego/Larissa/Bruno, [09:28] Bruno, [09:30] Bruno
