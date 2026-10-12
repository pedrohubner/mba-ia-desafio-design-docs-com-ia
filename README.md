# Da Reunião ao Documento: Design Docs do Sistema de Webhooks

> Enunciado original do desafio: [devfullcycle/mba-ia-desafio-design-docs-com-ia](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia#readme).
> Este README documenta **como** o pacote de design docs foi produzido.

## Sobre o desafio

Uma empresa que opera um OMS (Node.js + TypeScript + Express + Prisma + MySQL) decidiu, numa reunião de ~55 minutos, construir um **sistema de webhooks outbound** para avisar clientes B2B quando o status de um pedido muda. O que foi decidido ficou registrado só na transcrição da call (`TRANSCRICAO.md`). A tarefa foi transformar essa transcrição, junto com o código existente, num pacote de design docs acionável: PRD, RFC, FDD, ADRs e um Tracker de rastreabilidade. Tudo isso **sem alterar uma linha do código** da aplicação.

O ponto central foi o rigor de origem. Cada requisito, decisão ou restrição precisa apontar para uma fala (`[hh:mm] Nome`) ou para um arquivo real do repositório. Isso exige separar, na transcrição, o que foi **decidido**, o que foi **descartado**, o que foi **adiado** e o que é **detalhe secundário**. Também exige perceber os pontos que a reunião **não fechou**, mas que aparecem quando as decisões são combinadas.

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
|---|---|
| **Claude Code** (app desktop, modelo Claude Opus 5.5) | Ferramenta principal: leu o repositório e a transcrição, montou o mapa de decisões e redigiu todos os documentos |
| **Subagentes do Claude Code** (`general-purpose`) | Dois revisores adversariais rodando em paralelo: um caçando invenções e citações erradas, outro verificando escopo, fronteira entre documentos e o checklist do desafio |
| **Plugin `adrs-management`** (marketplace Full Cycle do curso) | O formato MADR do agente `adr-generator` (cabeçalho Status/Data/Relacionados; direcionadores; prós e contras; referências a arquivos) serviu de base para os ADRs, adaptado para incluir as seções exigidas pelo desafio |
| **Scripts Python gerados pela IA** | Validação automática: formato do tracker, se cada `[hh:mm] Nome` corresponde a uma fala real naquele minuto, se os caminhos de código existem e cobertura de IDs |

## Workflow adotado

A ordem seguiu a sugestão do enunciado: **decisões primeiro, consolidação de produto por último.**

```
1. Exploração        → leitura integral de TRANSCRICAO.md e do código que a feature toca
2. Mapa da reunião   → tabela interna: decidido / descartado / adiado / gancho no código / detalhe secundário
3. ADRs (7)          → uma decisão fechada por arquivo, com citações e referências de código
4. RFC               → proposta em nível de arquitetura, alternativas descartadas, questões em aberto
5. FDD               → modelo de dados, fluxos, contratos, erros, resiliência, observabilidade, integração
6. PRD               → consolidação em linguagem de produto (por quê / o quê)
7. Tracker           → uma linha por item com ID nos documentos
8. Validação         → script de checagem + 2 revisores adversariais em paralelo
9. Correções         → aplicação dos achados e nova rodada de validação
10. README           → este documento
```

Como a interação com a IA foi organizada:

- **Contexto completo antes de escrever.** A IA leu a transcrição inteira e os arquivos que a feature toca (`order.service.ts`, `order.status.ts`, `schema.prisma`, middlewares, erros, logger, `server.ts`, `app.ts`, testes). Só depois começou a redação, sem resumos intermediários que pudessem perder nuance.
- **IDs desde o primeiro rascunho.** Cada item recebeu um ID (`PRD-FR-03`, `FDD-ERR-07`, `RFC-ALT-02`...) já no documento, o que deixou o tracker verificável por script.
- **Separação entre "decidido" e "proposto".** O que a reunião não fixou e o FDD precisou escolher foi rotulado como *proposta de implementação* no texto e como `Proposta de implementação (derivada)` no tracker. Lacunas viraram a seção 5.2 do RFC.
- **Revisão por agentes independentes.** Os revisores receberam só as fontes e os documentos, sem o raciocínio de quem escreveu, com instrução explícita de não editar e de reportar apenas o que confirmassem na fonte.

## Prompts customizados

**1. Revisor adversarial de rastreabilidade** (subagente, executado em paralelo ao prompt 2):

```text
Você é um auditor técnico adversarial. NÃO edite nenhum arquivo — apenas leia e reporte.

Fontes de verdade: TRANSCRICAO.md (formato `[hh:mm] Nome: fala`) e o código em src/, prisma/,
tests/, package.json, docker-compose.yml. Documentos sob auditoria: docs/PRD.md, docs/RFC.md,
docs/FDD.md, docs/adrs/ADR-*.md.

Regra do desafio: toda informação deve ser rastreável à transcrição ou ao código. É proibido
inventar requisitos, decisões ou restrições. Itens rotulados como "proposta de implementação",
"provisório" ou listados como lacuna (L-01..L-06 no RFC) são permitidos — mas sinalize se algum
deles estiver apresentado como se fosse decisão da reunião.

1. Citações erradas: para CADA `[hh:mm] Nome`, confira se aquele falante disse aquilo naquele minuto.
2. Invenções: números, nomes, decisões ou restrições sem origem e sem rótulo de proposta/lacuna.
3. Contradições com a transcrição (algo descartado tratado como requisito; valores diferentes de
   2s, 10s, 64KB, 24h, 1m/5m/30m/2h/12h, 5 tentativas, 100 entregas) ou com o código (nome de
   método/classe/arquivo errado, comportamento de middleware descrito errado).
4. Todo caminho src/…, prisma/…, tests/… citado: verifique se existe. Inexistente sem "(novo)" = defeito.
5. Símbolos citados como existentes (changeStatus, requireRole, AppError, paginated,
   createPrismaClient, redactPaths, TxClient, validate, errorMiddleware…) existem no arquivo indicado?

Formato: CITAÇÕES ERRADAS (tabela) | INVENÇÕES | CONTRADIÇÕES | CAMINHOS/SÍMBOLOS INEXISTENTES | OK.
Só reporte o que você confirmou lendo a fonte.
```

**2. Revisor de escopo, fronteira entre documentos e checklist** (subagente):

```text
Você é um revisor de design docs exigente. NÃO edite nenhum arquivo — apenas leia e reporte.

Tarefa A — Filtro do que NÃO entra. A reunião descartou ou adiou: webhooks inbound, e-mail ao
cliente em falhas, rate limiting, dashboard, arquivamento (~30 dias), múltiplos workers / ordering
global, exactly-once, retry indefinido, 3 tentativas, secret global, disparo síncrono, Redis
Streams, trigger no banco, render do payload no envio, items no payload, endurecimento de RBAC.
Verifique se ALGUM aparece como requisito, objetivo, critério de aceite ou comportamento. Verifique
também se algo DECIDIDO na reunião ficou de fora (cobertura).

Tarefa B — Fronteira. PRD = por que/o quê; RFC = arquitetura concisa (2–4 páginas), alternativas e
questões em aberto; ADR = uma decisão; FDD = como construir. Aponte trechos do RFC no nível do FDD,
trechos técnicos demais no PRD, duplicação quase literal, e estime as páginas do RFC (wc -w).

Tarefa C — Checklist do desafio item a item (PASSA/FALHA com evidência) para PRD, RFC, FDD e ADRs,
incluindo todos os links markdown relativos (arquivos e âncoras estilo GitHub).

Tarefa D — Até 8 pontos genéricos, vagos, inconsistentes entre documentos, ou consequências
técnicas relevantes das decisões que não foram percebidas.
```

**3. Diretriz de produção usada na redação dos ADRs** (adaptada do agente `adr-generator` do plugin do curso):

```text
Para cada uma das decisões fechadas na reunião, gere um ADR em docs/adrs/ADR-NNN-titulo-kebab.md:
- Cabeçalho: Status, Data, ADRs relacionados. Seções: Status, Contexto, Direcionadores da Decisão,
  Alternativas Consideradas (máx. 3, ao menos 1 REAL citada na reunião), Decisão, Prós e Contras
  das Alternativas, Consequências (positivas, negativas e uma linha "Trade-off explícito"), Referências.
- Toda afirmação factual termina com a citação [hh:mm] Nome da fala de origem.
- Referências: no máximo 5 arquivos REAIS do repositório que a decisão toca.
- Se a reunião deixou um detalhe ambíguo (ex.: "5 tentativas" com 5 intervalos), NÃO escolha em
  silêncio: registre a interpretação como nota e leve a dúvida para "Questões em aberto" do RFC.
- Não transforme em decisão nada que foi descartado ou adiado.
```

## Iterações e ajustes

Foram **4 iterações principais** até o resultado final:

| # | Iteração | O que estava errado ou superficial | Ajuste |
|---|---|---|---|
| 1 | Geração inicial (ADRs → RFC → FDD → PRD) | Uma leitura literal da transcrição levaria a inconsistências. Elas foram percebidas e resolvidas na própria redação, sem passar em silêncio para os documentos (detalhes abaixo). | Interpretações explícitas e lacunas no RFC §5.2 |
| 2 | Primeira montagem do Tracker | Componentes, riscos e restrições do RFC **não tinham ID** no documento, então o tracker não tinha como apontar para eles | IDs `RFC-PROP-*`, `RFC-CTX-*`, `RFC-RISK-*`, `RFC-PLAN-01` inseridos no RFC. A edição automática quebrou os separadores das linhas de risco, o que foi detectado e corrigido. |
| 3 | Validação automática por script | Encontrou **14 caminhos de arquivo inexistentes** (os arquivos novos da feature) citados em alguns trechos sem a marcação "(novo)". Isso violaria o critério "nenhum arquivo mencionado é inexistente". Citações com intervalo (`[09:40]–[09:41] Bruno/Diego`) davam falso positivo no primeiro script. | Marcação "(novo)" em todas as ocorrências. Script refeito para validar intervalos. Resultado final: 100% das citações batem com falante e minuto. |
| 4 | Revisão adversarial (2 subagentes em paralelo) e correções | Cerca de 40 achados confirmados: citações distorcidas, propostas apresentadas como decisão, contradições com o código e defeitos de design (lista abaixo) | Correções em todos os documentos, tracker atualizado (295 linhas) e nova rodada dos scripts: 0 citações erradas, 0 links quebrados, 100% dos IDs rastreados |

**Problemas de conteúdo detectados e corrigidos na redação (iteração 1):**

- **"5 tentativas" vs. 5 intervalos.** A reunião fala em "5 tentativas" e lista cinco esperas (1m/5m/30m/2h/12h) somando "quase 15 horas". Com 5 envios no total, só 4 esperas seriam usadas (~2h36min). A única leitura coerente é 1 envio + 5 retentativas. A interpretação ficou explícita no ADR-003 e virou a lacuna `RFC-L-01` para o Diego confirmar.
- **Erro de validação vs. código `WEBHOOK_INVALID_URL`.** A Sofia pediu "erro de validação no schema Zod" e o Bruno citou o código `WEBHOOK_INVALID_URL`. Lendo `validate.middleware.ts`, todo `ZodError` vira `VALIDATION_ERROR`. Para não alterar o middleware compartilhado (decisão de reuso, ADR-006), o código específico vai em `details`. O trade-off está registrado no ADR-006 e no FDD.
- **Onde checar os 64KB.** A primeira ideia era checar o tamanho na inserção da outbox, onde o payload é renderizado. Só que, dentro da transação, um payload grande daria **rollback na mudança de status**, recriando o acoplamento que a outbox existe para eliminar. A checagem foi para o worker, com DLQ imediata.
- **Head-of-line blocking.** "Um único worker" processando em sequência, com timeout de 10s, deixaria um cliente lento atrasar todos os outros e estourar a meta de 10s. O FDD propõe concorrência por grupo de `orderId`, que preserva a ordem por pedido decidida na reunião.
- **Ordem sob retry.** Com backoff, um evento mais novo do mesmo pedido pode ultrapassar um evento em espera, o que contraria a "ordem por `order_id`" na prática. Isso ficou registrado como lacuna `RFC-L-04`, sem inventar solução.
- **FK da outbox para `orders`.** Uma FK impediria `DELETE /orders/:id` de pedidos `CANCELLED` com eventos, mudando o comportamento atual. A outbox guarda `orderId` sem FK, o que é coerente com o snapshot.
- **Lacunas de segurança.** Ficaram para a revisão da Sofia, sem decisões inventadas: assinatura durante o grace de 24h (`RFC-L-02`), `X-Timestamp` fora do conteúdo assinado (`RFC-L-03`) e armazenamento da secret (`RFC-L-06`).

**Achados da revisão adversarial (iteração 4).** Os mais relevantes e o que foi feito:

- **Citações que pareciam corretas, mas distorciam a fala:**
  - O FDD dizia "`customerId` no body ou na **query**" citando a Larissa, mas ela disse "body ou **path**". A listagem virou `GET /customers/:customerId/webhooks`.
  - A redaction de secrets no logger foi atribuída ao Diego, mas ele falou de um vazamento no log **do cliente**. Ficou como proposta do FDD.
  - "Estado ativo" foi atribuído à Sofia, mas quem propôs foi o Bruno.
  - O PRD trazia "cara para todos", mas o Marcos disse "cara para eles".
  - Os cenários atribuíam incidentes à Nova Cargo e à MaxDistribuição, mas a reunião não nomeou os clientes. Os cenários ficaram genéricos.
- **Propostas apresentadas como decisão:**
  - "5 retentativas" aparecia como fato no PRD e no título do ADR-003. O revisor achou **contraevidência** que eu tinha deixado passar: o resumo final diz "total 5 tentativas" ([09:48] Larissa). A RFC-L-01 foi reescrita com os dois lados.
  - Percentil p95, limiares de alerta, prefixo `sha256=`, detalhes do replay (202/409, zerar tentativas) e a exclusão do evento de criação de pedido passaram a ser rotulados como proposta ou interpretação. A exclusão também virou uma nova lacuna, a RFC-L-07.
- **Contradições com o código:**
  - O ADR-006 dizia que os erros do módulo estenderiam `NotFoundError`. Só que essa classe fixa o código `NOT_FOUND` no construtor, então nunca emitiria `WEBHOOK_NOT_FOUND`.
  - O caminho de `AppError` estava errado.
  - O enum `OrderStatus` foi atribuído a `order.status.ts`, mas vem do Prisma.
- **Defeitos de design que só apareceram com olhar externo:**
  - **Ordem.** A ordem por pedido era chamada de "garantida" e, em outro trecho, admitida como quebrável sob retry. O texto foi unificado.
  - **Cadência.** A cadência "processa o lote e dorme 2s" fazia um cliente lento segurar a detecção de novos eventos. O polling passou a não esperar envios em voo.
  - **Histórico.** "Últimos 100 webhooks enviados" tinha virado "100 tentativas". Passou a ser 100 eventos, com as tentativas aninhadas.
  - **Desativação.** Desativar um webhook mandava eventos para a DLQ e disparava alertas. Virou uma pausa real.
  - **DELETE.** O DELETE em cascata apagava a DLQ, junto com a evidência e a trilha de auditoria. A DLQ passou a ficar sem FK.
  - **Worker único.** Nada impedia dois workers num deploy com sobreposição. Entrou um lock nomeado do MySQL (`GET_LOCK`) e o risco FDD-RISK-09.
- **Rejeitado conscientemente:** a duplicação de "fora de escopo" entre PRD e FDD foi mantida, porque o desafio exige a seção nos dois. O cronograma duplicado foi removido do PRD, que agora aponta para o RFC §3.4.

## Como navegar a entrega

| Ordem | Documento | O que responde |
|---|---|---|
| 1 | [`docs/PRD.md`](docs/PRD.md) | **Por que e o quê.** Problema, público, métricas, escopo e fora de escopo, requisitos |
| 2 | [`docs/RFC.md`](docs/RFC.md) | **Como pretendemos resolver.** Proposta de arquitetura, alternativas descartadas, questões em aberto e lacunas |
| 3 | [`docs/adrs/`](docs/adrs/README.md) | **Por que exatamente assim.** 7 decisões: [001 Outbox](docs/adrs/ADR-001-outbox-transacional-no-mysql.md) · [002 Worker/polling](docs/adrs/ADR-002-worker-separado-em-polling.md) · [003 Retry/DLQ](docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) · [004 HMAC](docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) · [005 At-least-once](docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) · [006 Reuso de padrões](docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) · [007 Snapshot/filtro](docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md) |
| 4 | [`docs/FDD.md`](docs/FDD.md) | **Como construir.** Modelo de dados, fluxos, 8 contratos, matriz `WEBHOOK_*`, resiliência, observabilidade e integração com 18 pontos do código existente |
| 5 | [`docs/TRACKER.md`](docs/TRACKER.md) | **De onde veio cada coisa.** Uma linha por item com ID, apontando para `[hh:mm] Nome` ou para um arquivo |

Sugestão: quem vai **implementar** pode ir direto do RFC §3 para o FDD §10 (integração) e §5 (fluxos). Quem vai **revisar** deve começar pelo RFC §5 (questões em aberto e lacunas).

```
.
├── README.md                 ← este documento (processo)
├── TRANSCRICAO.md            ← fonte (inalterada)
├── docs/
│   ├── PRD.md
│   ├── RFC.md
│   ├── FDD.md
│   ├── TRACKER.md
│   └── adrs/
│       ├── README.md         ← índice dos ADRs
│       └── ADR-001 … ADR-007
└── src/ prisma/ tests/       ← código da aplicação (inalterado)
```
