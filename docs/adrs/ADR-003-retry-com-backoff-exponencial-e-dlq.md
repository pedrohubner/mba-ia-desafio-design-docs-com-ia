# ADR-003: Retry com backoff exponencial (1m/5m/30m/2h/12h) e DLQ em tabela separada

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Status

Aceito. Retry decidido em [09:17] por Larissa e DLQ com replay administrativo em [09:18]–[09:19]. Role ADMIN no replay decidida em [09:36]. Tudo confirmado no resumo de [09:48].

## Contexto

O endpoint do cliente pode estar lento, fora do ar ou em manutenção planejada. Já houve cliente com duas horas de indisponibilidade ([09:16] Diego). O worker ([ADR-002](ADR-002-worker-separado-em-polling.md)) precisa decidir o que fazer quando uma entrega falha, sem desistir cedo demais e sem deixar o evento "pendurado para sempre" ([09:15] Diego).

Uma tentativa conta como falha quando o cliente não responde em **10 segundos** (timeout do HTTP call, [09:42] Diego) ou quando a entrega não tem sucesso.

## Direcionadores da Decisão

- Cobrir indisponibilidades de algumas horas do cliente ([09:16] Diego).
- Ter fim determinístico: nenhum evento em retry infinito ([09:15] Diego).
- Guardar evidência para debug e reprocessamento ([09:18] Diego).
- Manter limpa a leitura da outbox principal ([09:18] Diego).

## Alternativas Consideradas

1. **Retry indefinido com backoff** ([09:15] Diego, citando quem defende).
2. **3 tentativas**, mais agressivo ([09:16] Bruno).
3. **5 tentativas com backoff 1m / 5m / 30m / 2h / 12h e DLQ em tabela separada** ([09:15]–[09:18] Diego).

Para onde vão os eventos esgotados, também se comparou **marcar como `failed` na própria outbox** com **tabela separada `webhook_dead_letter`** ([09:17] Larissa, [09:18] Diego).

## Decisão

Adotamos a **alternativa 3**:

- **Backoff exponencial** com intervalos de **1 min, 5 min, 30 min, 2 h e 12 h** e "5 tentativas" ([09:17] Diego, decidido em [09:17] Larissa). Entre a primeira falha e a última tentativa passam cerca de 14h36min, o "quase 15 horas" de [09:17] Diego.
  - *Nota de interpretação:* a reunião fala em "5 tentativas" e lista **5 intervalos**. Os dois só fecham com as "quase 15 horas" se forem 5 retentativas **depois** da tentativa inicial (1 + 5 envios HTTP). Com 5 envios no total, só 4 intervalos seriam usados (≈2h36min). Há contraevidência: o resumo final diz "total 5 tentativas" ([09:48] Larissa). Este ADR e o FDD adotam a primeira leitura, porque é a única que usa os cinco intervalos decididos. A confirmação está pendente na RFC-L-01 do [RFC](../RFC.md#5-questões-em-aberto).
- Esgotadas as retentativas, o evento é **falha permanente** e vai para a **DLQ**, uma tabela separada `webhook_dead_letter` com o payload, o motivo da falha e o timestamp ([09:18] Diego). Na outbox, o evento fica no estado terminal "falhou", um dos quatro estados de [09:08] Diego, e sai da leitura do worker.
- **Reprocessamento manual** por endpoint administrativo `POST /admin/webhooks/dead-letter/:id/replay`, que **recoloca o evento na outbox como pendente** ([09:18] Diego).
- O replay exige **role ADMIN**, reaproveitando o `requireRole` existente, e **registra quem fez o replay**, para auditoria ([09:36] Sofia, [09:36] Larissa).

## Prós e Contras das Alternativas

### Retry indefinido
- ✅ Nunca desiste de entregar.
- ❌ Evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego).
- ❌ A outbox acumula lixo sem fim.

### 3 tentativas
- ✅ Libera a fila rápido.
- ❌ Desistiria em cerca de 30 minutos, menos do que uma manutenção planejada de 2h já vista em produção ([09:16] Diego).

### 5 tentativas com backoff + DLQ separada
- ✅ Cobre janela de quase 15h, considerada aceitável pelo produto ([09:17] Marcos).
- ✅ A DLQ separada mantém a outbox enxuta e funciona como evidência ([09:18] Diego).
- ❌ Uma nova tabela e um endpoint administrativo para manter.
- ❌ Depois de esgotado, o evento só volta com intervenção humana.

## Consequências

**Positivas:** o comportamento de falha é previsível e tem fim. Indisponibilidades de até ~14h36min se recuperam sem intervenção. Falhas permanentes ficam com payload e motivo preservados, e o reprocessamento é auditável.

**Negativas:** um cliente fora do ar por mais de ~15h perde a entrega automática e depende de replay manual por um ADMIN. A reunião adiou a notificação proativa do cliente quando o webhook dele falha (por exemplo, e-mail após falhas seguidas) para uma próxima fase ([09:37] Larissa). Com single-worker, um evento em backoff pode ser ultrapassado por um evento mais novo do mesmo pedido (ver questão em aberto no [RFC](../RFC.md#5-questões-em-aberto)).

**Trade-off explícito:** priorizamos um fim determinístico e evidência auditável em vez de tentar entregar para sempre.

## Referências

- `src/middlewares/auth.middleware.ts` (`requireRole`, reaproveitado no replay)
- `src/modules/users/user.routes.ts` (exemplo existente de rota protegida com `requireRole('ADMIN')`)
- `src/shared/logger/index.ts` (logger Pino usado no log de auditoria do replay)
- Transcrição: [09:14]–[09:19] Larissa/Diego/Bruno/Marcos, [09:35]–[09:36] Larissa/Sofia, [09:42] Diego
