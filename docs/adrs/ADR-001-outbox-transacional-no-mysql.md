# ADR-001: Padrão Outbox transacional no MySQL existente

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-snapshot-do-payload-e-filtro-na-insercao.md)

## Status

Aceito. Decidido por Larissa (Tech Lead) em [09:08], com a proposta de Diego em [09:06] e a confirmação final no resumo de [09:48].

## Contexto

Três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) querem saber em tempo quase real quando o status dos pedidos deles muda. Hoje fazem polling em `GET /orders` ([09:00] Marcos). O evento de origem é a mudança de status de pedido, que já roda numa transação pesada: no método `changeStatus` de `src/modules/orders/order.service.ts`, ela atualiza `orders`, insere em `order_status_history` e debita ou repõe `stock_quantity` ([09:04] Bruno).

O problema tem duas faces. Se a notificação HTTP for disparada dentro dessa transação, qualquer cliente lento trava a mudança de status de outros pedidos, e um cliente fora do ar não justifica rollback de uma mudança de status válida ([09:04] Bruno). Se a notificação for disparada fora da transação, sem nenhum registro durável, um crash entre o commit e o envio perde o evento. O requisito explícito é que "não pode ter caso de status mudar e evento não sair" ([09:40] Bruno).

O time é pequeno e a stack atual tem só Node.js, Express, Prisma e MySQL, sem fila nem broker (`package.json`, `docker-compose.yml`).

## Direcionadores da Decisão

- Atomicidade: o evento existe se, e somente se, a mudança de status commitou ([09:06] Diego, [09:40] Bruno).
- Isolamento: latência ou indisponibilidade do cliente não pode afetar a API de pedidos ([09:04] Bruno).
- Custo operacional: não subir infraestrutura nova para um time pequeno ([09:07] Diego).
- Latência aceitável abaixo de 10 segundos ([09:02] Marcos).

## Alternativas Consideradas

1. **Disparo síncrono no `OrderService.changeStatus`**: a chamada HTTP acontece dentro do fluxo de mudança de status.
2. **Redis Streams (ou fila equivalente)**: publicar o evento num broker externo ([09:07] Larissa).
3. **Outbox transacional no MySQL existente**: inserir uma linha em `webhook_outbox` na mesma transação SQL da mudança de status; um worker separado lê e entrega ([09:06] Diego).

## Decisão

Adotamos a **alternativa 3**: o padrão Outbox numa tabela `webhook_outbox` do mesmo banco MySQL, gravada dentro da mesma transação Prisma (`prisma.$transaction`) que já protege a mudança de status. Se a inserção na outbox falhar, a transação inteira dá rollback ([09:40] Bruno). A tabela tem índices em status do evento e em `created_at`, para o worker ler só os pendentes em lotes pequenos ([09:08] Diego). O consumo da outbox está descrito no [ADR-002](ADR-002-worker-separado-em-polling.md).

O arquivamento das linhas já entregues (por exemplo, depois de cerca de 30 dias) **fica fora do escopo** desta feature ([09:08] Diego).

## Prós e Contras das Alternativas

### Disparo síncrono
- ✅ Implementação mais simples e latência mínima.
- ❌ "Qualquer cliente lento vai travar mudança de status pra outros pedidos" ([09:04] Bruno).
- ❌ Com o cliente fora do ar, só sobram duas saídas ruins: dar rollback numa mudança de status válida ou perder o evento ([09:04] Bruno).
- ❌ Descartada por unanimidade: "Síncrono está fora de questão" ([09:06] Diego).

### Redis Streams
- ✅ Feito para mensageria e mais reativo que polling.
- ❌ Exige infraestrutura nova (Redis Cluster), considerada overengineering para o time ([09:07] Diego).
- ❌ *(Análise nossa, não discutida na reunião.)* Sem transação distribuída, volta o problema de atomicidade entre MySQL e o broker.

### Outbox no MySQL
- ✅ Atomicidade garantida pela própria transação do banco ([09:06] Diego).
- ✅ Zero infraestrutura nova: reusa banco, Prisma e pool de conexões.
- ❌ Introduz latência de polling e carga de leitura sobre o MySQL (ver [ADR-002](ADR-002-worker-separado-em-polling.md)).
- ❌ A tabela cresce sem limite enquanto não existir uma rotina de arquivamento.

## Consequências

**Positivas:** a mudança de status e o registro do evento passam a ser uma unidade atômica. Não existe estado em que o pedido mudou e o evento sumiu, nem evento de uma mudança que deu rollback. A disponibilidade do cliente deixa de afetar a API de pedidos, porque o HTTP sai do caminho crítico.

**Negativas:** a transação de `changeStatus` ganha mais uma escrita (uma linha por webhook interessado, ver [ADR-007](ADR-007-snapshot-do-payload-e-filtro-na-insercao.md)). O MySQL vira também uma fila, com a carga de polling e o crescimento da tabela que isso traz. O arquivamento foi explicitamente adiado e precisa ser acompanhado como dívida.

**Trade-off explícito:** aceitamos latência de alguns segundos e uma tabela que cresce continuamente em troca de garantia transacional sem infraestrutura nova.

## Referências

- `src/modules/orders/order.service.ts` (método `changeStatus`, transação Prisma)
- `prisma/schema.prisma` (modelos `Order` e `OrderStatusHistory`)
- `docker-compose.yml` (infraestrutura atual: só MySQL 8.0)
- Transcrição: [09:03] Larissa, [09:04] Bruno, [09:06] Diego, [09:07] Larissa/Diego, [09:08] Diego/Larissa, [09:40]–[09:41] Bruno/Diego
