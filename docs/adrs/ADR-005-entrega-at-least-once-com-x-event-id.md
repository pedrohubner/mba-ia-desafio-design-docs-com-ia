# ADR-005: Garantia de entrega at-least-once com deduplicação pelo cliente via `X-Event-Id`

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-001](ADR-001-outbox-transacional-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-004](ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)

## Status

Aceito. Decidido por Larissa em [09:26] e confirmado no resumo de [09:48].

## Contexto

A combinação de outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)) com retry ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) pode gerar entregas duplicadas. Um exemplo: o cliente processa o evento, mas a resposta não volta dentro do timeout de 10s. O worker marca falha e reenvia. Outro: o worker cai depois do envio e antes de gravar o sucesso. Também o replay manual da DLQ reenvia um evento.

Garantir exatamente uma entrega exigiria coordenação entre os dois lados ([09:25] Diego).

## Direcionadores da Decisão

- Nenhum evento perdido, o que é mais importante do que evitar duplicatas ([09:40] Bruno).
- Simplicidade de implementação para um time pequeno.
- Alinhamento com o mercado: Stripe e GitHub usam o mesmo modelo ([09:25] Diego).
- Dar ao cliente um meio determinístico de deduplicar ([09:25] Bruno/Diego).

## Alternativas Consideradas

1. **Exactly-once**, com coordenação entre emissor e receptor ([09:25] Diego).
2. **At-most-once**, sem retry. Não foi discutida diretamente e é incompatível com o [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md).
3. **At-least-once com identificador único de evento no header `X-Event-Id`** ([09:24]–[09:25] Diego).

## Decisão

Adotamos a **alternativa 3**:

- A plataforma garante **at-least-once**: o cliente **pode receber o mesmo evento mais de uma vez** e precisa estar preparado ([09:24] Diego).
- Todo envio carrega o header **`X-Event-Id`** com um **UUID gerado quando o evento entra na outbox**, único por evento ([09:25] Diego). O mesmo valor aparece no campo `event_id` do payload ([09:43] Diego).
- O **cliente deduplica** pelo `event_id` do lado dele ([09:25] Diego).
- O produto documenta esse contrato **em destaque no portal do desenvolvedor** ([09:26] Marcos).
- O identificador segue o padrão do projeto: tudo é UUID ([09:51] Larissa).

## Prós e Contras das Alternativas

### Exactly-once
- ✅ O cliente não precisa deduplicar.
- ❌ Exige coordenação dos dois lados e é muito mais complexo ([09:25] Diego).
- ❌ Depende de capacidades do lado do cliente que não controlamos.

### At-most-once
- ✅ Nunca duplica.
- ❌ Perde eventos em qualquer falha transitória, o que contraria o requisito central da feature.

### At-least-once + `X-Event-Id`
- ✅ Resolve "99% dos casos" com baixo custo ([09:25] Diego).
- ✅ Padrão já conhecido por integradores (Stripe, GitHub).
- ❌ Passa responsabilidade para o cliente ([09:25] Sofia).

## Consequências

**Positivas:** o mecanismo de entrega fica simples: não precisa de estado distribuído nem de protocolo de confirmação. Retries e replays são seguros desde que o cliente deduplique. O `X-Event-Id` também serve como chave de correlação de ponta a ponta em logs e no histórico de entregas.

**Negativas:** um cliente que não implementar a deduplicação pode processar o mesmo evento duas vezes. A mitigação é documentação e comunicação, a cargo do PM ([09:26] Marcos), e não algo que a plataforma consiga impor.

**Trade-off explícito:** aceitamos duplicatas possíveis e transferimos a deduplicação ao cliente em troca de nunca perder evento e de manter o emissor simples.

## Referências

- `prisma/schema.prisma` (convenção de IDs `@default(uuid()) @db.Char(36)`)
- `src/middlewares/request-logger.middleware.ts` (precedente de identificador de correlação por UUID via `uuid`)
- `package.json` (dependência `uuid` já presente)
- Transcrição: [09:24]–[09:26] Diego/Bruno/Sofia/Marcos/Larissa, [09:43] Diego, [09:51] Larissa
