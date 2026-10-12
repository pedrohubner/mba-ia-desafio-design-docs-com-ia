# Architectural Decision Records

Este diretório guarda os ADRs (Architectural Decision Records) da feature **Sistema de Webhooks de Notificação de Pedidos**. Cada decisão arquitetural fechada na reunião técnica (`TRANSCRICAO.md`) tem um arquivo próprio, no formato `ADR-NNN-titulo-em-kebab-case.md`, numa variante do MADR em português.

| ADR | Decisão | Status |
|---|---|---|
| [ADR-001](ADR-001-outbox-transacional-no-mysql.md) | Padrão Outbox transacional no MySQL existente | Aceito |
| [ADR-002](ADR-002-worker-separado-em-polling.md) | Worker em processo separado, polling de 2s | Aceito |
| [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md) | Retry com backoff exponencial (1m/5m/30m/2h/12h) e DLQ em tabela separada | Aceito |
| [ADR-004](ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação com grace de 24h | Aceito |
| [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com deduplicação via `X-Event-Id` | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md) | Reuso máximo dos padrões existentes do projeto | Aceito |
| [ADR-007](ADR-007-snapshot-do-payload-e-filtro-na-insercao.md) | Payload enxuto como snapshot e filtro de eventos na inserção | Aceito |
