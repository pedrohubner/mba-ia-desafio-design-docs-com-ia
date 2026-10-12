# ADR-007: Payload enxuto renderizado como snapshot e filtro de eventos aplicado na inserção da outbox

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-001](ADR-001-outbox-transacional-no-mysql.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Status

Aceito. Filtro na inserção acordado em [09:34] (Bruno/Diego). Formato do payload definido em [09:43]–[09:44]. Snapshot decidido em [09:52] por Larissa/Diego/Bruno, numa conversa depois da saída de Marcos e Sofia.

## Contexto

Três decisões secundárias, ligadas entre si, definem **o que** vai para a outbox:

1. **Quando filtrar.** Cada webhook escolhe os status que quer ouvir, por exemplo "só SHIPPED e DELIVERED" ([09:33] Marcos). O filtro pode rodar na inserção da outbox ou na hora do envio ([09:34] Diego).
2. **Quando renderizar o payload.** A outbox pode guardar o payload pronto ou só o `order_id`, para montar o JSON na hora do envio ([09:51] Bruno). Como o envio pode acontecer horas depois, até ~15h com retries ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)), o pedido pode ter mudado nesse meio-tempo.
3. **Quanto incluir.** O pedido tem itens (`order_items`) e outros dados que podem inflar a mensagem. Há também um limite de 64KB por payload ([09:24] Diego).

## Direcionadores da Decisão

- O evento precisa refletir o estado **no momento da mudança** ([09:52] Larissa).
- Economizar linhas na outbox ([09:34] Bruno).
- Payload enxuto, longe do teto de 64KB ([09:43]–[09:44] Diego/Bruno).
- Detalhes ficam disponíveis por API já existente ([09:43] Diego).

## Alternativas Consideradas

1. **Filtrar no envio e renderizar no envio**: inserir sempre, guardar só a referência ao pedido e decidir e montar no worker.
2. **Filtrar na inserção e renderizar no envio.**
3. **Filtrar na inserção e guardar o payload renderizado (snapshot)** ([09:34] Bruno, [09:52] Larissa).

## Decisão

Adotamos a **alternativa 3**:

- **Filtro na inserção:** dentro da transação de mudança de status, só os webhooks **ativos** do customer que assinam o `to_status` recebem um evento na outbox. Se **nenhum** webhook do customer quer aquele status, **nada é inserido** ([09:34] Bruno, [09:34] Diego).
- **Snapshot na inserção:** o payload JSON é renderizado e gravado na outbox no momento da mudança. Mudanças posteriores no pedido não alteram eventos já enfileirados ([09:52] Larissa/Diego/Bruno).
- **Formato enxuto**, com tipo de evento `order.status_changed`: `event_id`, `event_type`, `timestamp` (ISO 8601), `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos da order, como `total_cents`. **Não inclui `items`.** Para detalhes, o cliente consulta `GET /orders/:id` ([09:43] Diego).

## Prós e Contras das Alternativas

### Filtrar e renderizar no envio
- ✅ *(Análise nossa.)* Transação de `changeStatus` mais leve.
- ❌ A outbox se enche de eventos que ninguém quer ([09:34] Bruno).
- ❌ O payload mostraria o estado atual do pedido, não o da mudança ("caso esquisito", [09:52] Larissa).

### Filtrar na inserção e renderizar no envio
- ✅ Outbox enxuta.
- ❌ O mesmo problema de consistência temporal do payload.

### Filtrar na inserção com snapshot
- ✅ O evento é um fato imutável, coerente com o que o cliente vê no histórico.
- ✅ O worker não precisa consultar `orders`, só a outbox e a configuração do webhook.
- ❌ A transação de `changeStatus` faz uma leitura a mais (webhooks do customer) e N inserções.
- ❌ Uma alteração de assinatura feita depois da mudança de status não alcança eventos já enfileirados (e vice-versa).

## Consequências

**Positivas:** o evento entregue é sempre fiel ao momento em que ocorreu, mesmo depois de horas de retry ou de um replay da DLQ. O worker fica simples. O payload pequeno mantém folga em relação ao limite de 64KB.

**Negativas:** a transação crítica de pedidos fica um pouco mais pesada. Ela já é "pesada", nas palavras de [09:04] Bruno. O cliente que precisar de itens ou endereço faz uma chamada extra a `GET /orders/:id`.

**Trade-off explícito:** aceitamos um pouco mais de trabalho dentro da transação em troca de eventos imutáveis e de uma outbox sem linhas inúteis.

## Referências

- `src/modules/orders/order.service.ts` (`changeStatus`: ponto de inserção, com `from`/`to` disponíveis)
- `src/modules/orders/order.status.ts` (máquina de estados que define as transições possíveis)
- `src/modules/orders/order.routes.ts` (`GET /orders/:id`, usado pelo cliente para obter detalhes)
- Transcrição: [09:33]–[09:34] Marcos/Bruno/Diego, [09:43]–[09:44] Diego/Bruno, [09:51]–[09:52] Bruno/Larissa/Diego
