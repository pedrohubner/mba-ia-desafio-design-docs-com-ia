# ADR-006: Reuso máximo dos padrões e da infraestrutura existentes do projeto

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-004](ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)

## Status

Aceito. Decidido por Larissa em [09:30], com o prefixo de erro fixado em [09:29], e confirmado no resumo de [09:48].

## Contexto

O OMS tem convenções claras e consistentes, que se veem no código:

- **Módulos por domínio** em `src/modules/<domínio>/`, com `*.controller.ts`, `*.service.ts`, `*.repository.ts`, `*.routes.ts` e `*.schemas.ts` (por exemplo, `src/modules/orders/` e `src/modules/customers/`) ([09:27] Bruno).
- **Hierarquia de erros** com a classe base `AppError` (`src/shared/errors/app-error.ts`) e especializações com código semântico em SCREAMING_SNAKE_CASE, como `InsufficientStockError` → `INSUFFICIENT_STOCK` e `InvalidStatusTransitionError` → `INVALID_STATUS_TRANSITION` (`src/shared/errors/http-errors.ts`) ([09:28] Bruno).
- **Middleware de erro centralizado** (`src/middlewares/error.middleware.ts`), que já traduz `AppError`, `ZodError` e erros conhecidos do Prisma em respostas HTTP ([09:29] Bruno).
- **Logger Pino** único (`src/shared/logger/index.ts`) ([09:29] Bruno).
- **Validação com Zod** via `validate()` (`src/middlewares/validate.middleware.ts`) e **autorização** via `authenticate`/`requireRole` (`src/middlewares/auth.middleware.ts`).

A feature de webhooks precisa de API REST, processamento assíncrono, erros próprios e logs. A dúvida era se ela seguiria essas convenções ou traria bibliotecas e estruturas novas.

## Direcionadores da Decisão

- Consistência com o padrão claro que a base já tem ([09:27] Bruno).
- Os erros novos precisam ser tratados pelo middleware existente sem nenhuma alteração nele ([09:29] Bruno).
- Nada de dependência nova para logging ([09:29] Bruno).
- Prazo apertado de três sprints ([09:46] Larissa).

## Alternativas Consideradas

1. **Módulo isolado com stack própria**: outro framework de validação ou de logs, ou um serviço à parte. É uma alternativa plausível, que a reunião rejeitou de forma implícita ao fixar o reuso.
2. **Reuso parcial**: seguir a estrutura de pastas, mas com tratamento de erro e códigos ad hoc dentro do módulo.
3. **Reuso máximo**: o webhook vira "módulo igual aos outros" ([09:30] Larissa).

## Decisão

Adotamos a **alternativa 3**. Concretamente:

| Padrão existente | Como o módulo de webhooks reutiliza | Origem |
|---|---|---|
| Estrutura de módulo (`src/modules/orders/`) | Pasta `src/modules/webhooks/` (nova) com controller, service, repository, routes e schemas | [09:27] Bruno |
| `AppError` + subclasses (`src/shared/errors/`) | Classes de erro do módulo estendem `AppError` (404 com código próprio) e `ConflictError` (que aceita código customizado). `NotFoundError` não serve, porque fixa o código `NOT_FOUND` no construtor. | [09:28] Bruno |
| Códigos semânticos (`INSUFFICIENT_STOCK`) | **Todos os códigos do módulo usam o prefixo `WEBHOOK_`** (por exemplo, `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`) | [09:28] Bruno, [09:29] Larissa |
| `errorMiddleware` | Usado sem alteração; os erros novos são `AppError` e já saem formatados | [09:29] Bruno |
| Logger Pino | Mesmo logger na API e no worker; nenhuma lib nova | [09:29] Bruno |
| Zod + `validate()` | Schemas em `webhook.schemas.ts` (novo), incluindo a regra `https` | [09:23] Sofia, [09:30] Larissa |
| `authenticate` / `requireRole` | CRUD com `authenticate`; replay com `requireRole('ADMIN')` | [09:36] Larissa |
| `createPrismaClient()` (`src/config/database.ts`) | O worker cria a **própria** instância (PrismaClient é por processo), com a mesma `DATABASE_URL` | [09:30] Bruno |
| IDs UUID (`prisma/schema.prisma`) | Todas as tabelas novas usam UUID | [09:51] Larissa |

## Prós e Contras das Alternativas

### Stack própria
- ✅ Liberdade de escolher ferramentas "ideais" para mensageria.
- ❌ Duas formas de fazer a mesma coisa no repositório e mais custo de manutenção.
- ❌ Contraria a orientação explícita de "não vamos botar nada novo" ([09:29] Bruno).

### Reuso parcial
- ✅ Começa rápido.
- ❌ Códigos e formatos de erro inconsistentes para os consumidores da API.

### Reuso máximo
- ✅ Respostas de erro e logs idênticos aos do resto da API.
- ✅ Middleware, auth e validação já testados.
- ❌ O módulo herda as limitações atuais, por exemplo: `validate()` converte qualquer `ZodError` em `VALIDATION_ERROR`, então um código `WEBHOOK_*` específico de validação só aparece em `details`.

## Consequências

**Positivas:** quem conhece `src/modules/orders/` navega em `src/modules/webhooks/` sem esforço. O `errorMiddleware` não muda. A observabilidade continua uniforme (mesmo logger, mesmos campos base `service`/`env`).

**Negativas:** a feature fica presa às capacidades atuais da base. Não existe biblioteca de métricas nem de tracing no projeto (`package.json`), então observabilidade avançada fica limitada a logs estruturados e consultas ao banco, como detalha o FDD. Há também acoplamento de deploy: API e worker compartilham código e versão.

**Trade-off explícito:** abrimos mão de ferramentas especializadas em troca de consistência, menos dependências e entrega no prazo.

## Referências

- `src/shared/errors/app-error.ts` (`AppError`) e `src/shared/errors/http-errors.ts` (`InsufficientStockError`, `InvalidStatusTransitionError`)
- `src/middlewares/error.middleware.ts` (tratamento centralizado de `AppError`, Zod e Prisma)
- `src/middlewares/auth.middleware.ts` (`authenticate`, `requireRole`)
- `src/shared/logger/index.ts` (logger Pino)
- `src/modules/orders/order.routes.ts` (padrão de rotas com `authenticate` + `validate`)
