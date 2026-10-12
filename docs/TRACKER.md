# Tracker de Rastreabilidade — Sistema de Webhooks de Notificação de Pedidos

Este tracker liga cada item registrado no pacote de documentos à sua **origem**: um momento da reunião (`TRANSCRICAO.md`, no formato `[hh:mm] Nome`) ou um arquivo do código-fonte existente.

**Como ler:**
- **ID** é o identificador que aparece no próprio documento. Itens de ADR usam `ADR-NNN` para a decisão e `ADR-NNN-ALT-n` para a alternativa *n* da seção "Alternativas Consideradas".
- **Tipo** `Proposta de implementação (derivada)` marca escolhas de detalhe feitas no FDD que **não foram fixadas na reunião**. A localização aponta para a fala ou o arquivo que motivou a escolha. Esses itens são candidatos naturais a ajuste na revisão de design.
- **Tipo** `Lacuna` marca pontos que a reunião deixou implícitos e que aparecem quando duas decisões são combinadas (RFC §5.2).
- IDs com sufixo (`-REF`, `a`/`b`, `-CFG`, etc.) apontam para subitens dentro da seção do ID pai no documento (por exemplo, `FDD-FLUXO-01a` é um passo do fluxo FDD-FLUXO-01; `ADR-006-REF1` é uma referência de código do ADR-006).
- Quando um item tem várias falas de origem, a coluna Localização mostra a **mais decisiva**. As demais estão citadas no corpo do documento.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CEN-01 | docs/PRD.md | Cenário | Cliente assina só SHIPPED/DELIVERED e recebe notificação em segundos | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-02 | docs/PRD.md | Cenário | Endpoint do cliente em manutenção de 2h; retentativas entregam depois | TRANSCRICAO | [09:16] Diego |
| PRD-CEN-03 | docs/PRD.md | Cenário | Cliente vaza secret e rotaciona com 24h de convivência | TRANSCRICAO | [09:22] Diego |
| PRD-CEN-04 | docs/PRD.md | Cenário | Cliente investiga entrega pelo histórico das últimas entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-CEN-05 | docs/PRD.md | Cenário | Cliente fora > 15h; evento na DLQ e replay por administrador | TRANSCRICAO | [09:18] Diego |
| PRD-CEN-06 | docs/PRD.md | Cenário | Cliente recebe duplicata e deduplica pelo identificador do evento | TRANSCRICAO | [09:25] Diego |
| PRD-MET-01 | docs/PRD.md | Métrica | p95 entre mudança de status e 1ª tentativa < 10s | TRANSCRICAO | [09:02] Marcos |
| PRD-MET-02 | docs/PRD.md | Métrica | Detecção da mudança em ≤ 2s (polling) | TRANSCRICAO | [09:10] Larissa |
| PRD-MET-03 | docs/PRD.md | Métrica | Zero mudanças de status sem notificação | TRANSCRICAO | [09:40] Bruno |
| PRD-MET-04 | docs/PRD.md | Métrica | Janela de ~14h36min de indisponibilidade sem intervenção | TRANSCRICAO | [09:17] Diego |
| PRD-MET-05 | docs/PRD.md | Métrica | 3 de 3 clientes integrados; Atlas até fim de novembro | TRANSCRICAO | [09:45] Marcos |
| PRD-MET-06 | docs/PRD.md | Métrica | Desenvolvimento em 3 sprints, com revisão de segurança | TRANSCRICAO | [09:47] Larissa |
| PRD-MET-07 | docs/PRD.md | Métrica | Acompanhar a queda do polling em GET /orders (sem meta) | TRANSCRICAO | [09:00] Marcos |
| PRD-MET-08 | docs/PRD.md | Métrica | Taxa de falha e volume de DLQ por cliente, como insumo da fase 2 | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-01 | docs/PRD.md | Fora de escopo | E-mail ao cliente em falhas repetidas (próxima fase) | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-02 | docs/PRD.md | Fora de escopo | Dashboard visual (projeto do frontend) | TRANSCRICAO | [09:40] Larissa |
| PRD-OOS-03 | docs/PRD.md | Fora de escopo | Rate limiting de saída (observar e decidir depois) | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-04 | docs/PRD.md | Fora de escopo | Webhooks inbound | TRANSCRICAO | [09:02] Marcos |
| PRD-OOS-05 | docs/PRD.md | Fora de escopo | Arquivamento de entregues (~30 dias) | TRANSCRICAO | [09:08] Diego |
| PRD-OOS-06 | docs/PRD.md | Fora de escopo | Ordem global e múltiplos workers | TRANSCRICAO | [09:13] Diego |
| PRD-OOS-07 | docs/PRD.md | Fora de escopo | Exactly-once | TRANSCRICAO | [09:25] Diego |
| PRD-OOS-08 | docs/PRD.md | Fora de escopo | Restrição de role no CRUD (endurecer depois) | TRANSCRICAO | [09:37] Sofia |
| PRD-OOS-09 | docs/PRD.md | Fora de escopo | Itens do pedido no payload | TRANSCRICAO | [09:43] Diego |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook (URL + status); cliente explícito, não vem do JWT | TRANSCRICAO | [09:32] Larissa |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Secret gerada pela plataforma e devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Listar webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Editar webhook (URL, status, ativo) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Remover webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Filtro por status; sem interessado, nada é gerado | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Notificação `order.status_changed` com os campos definidos | TRANSCRICAO | [09:43] Diego |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Assinatura para comprovar origem e integridade | TRANSCRICAO | [09:19] Sofia |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Nova secret por API; antiga válida por 24h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Retentativas com espera 1m/5m/30m/2h/12h (contagem em RFC-L-01) | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | Fila de não entregues com conteúdo, motivo e momento | TRANSCRICAO | [09:18] Diego |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Replay por administrador, registrando o autor | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-13 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas (sucesso, payload, resposta, tempo) | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-14 | docs/PRD.md | Requisito Funcional | Identificador do evento e do webhook em cada notificação | TRANSCRICAO | [09:44] Sofia |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Latência < 10s; mínima de até 2s aceita | TRANSCRICAO | [09:10] Larissa |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Atomicidade entre mudança de status e notificação | TRANSCRICAO | [09:06] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Isolamento: cliente lento não afeta mudanças de status | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | At-least-once com deduplicação pelo cliente | TRANSCRICAO | [09:26] Larissa |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | Ordem só por pedido e com um único processador | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-06 | docs/PRD.md | Restrição | Só URLs https; http recusada | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-07 | docs/PRD.md | Restrição | Secret única por webhook, nunca global | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-08 | docs/PRD.md | Restrição | Payload > 64KB não é enviado; erra, não trunca | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Timeout de 10s por chamada | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-10 | docs/PRD.md | Requisito Não Funcional | Envio separado da API, sobrevive a restart da API | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-11 | docs/PRD.md | Restrição | Sem infraestrutura nova | TRANSCRICAO | [09:07] Diego |
| PRD-NFR-12 | docs/PRD.md | Requisito Não Funcional | Auditoria do replay (quem executou) | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-13 | docs/PRD.md | Requisito Não Funcional | Conteúdo enxuto e congelado no momento da mudança | TRANSCRICAO | [09:52] Larissa |
| PRD-DEC-01 | docs/PRD.md | Decisão | Outbox no banco atual | TRANSCRICAO | [09:08] Larissa |
| PRD-DEC-02 | docs/PRD.md | Decisão | Processador separado com polling de 2s | TRANSCRICAO | [09:10] Larissa |
| PRD-DEC-03 | docs/PRD.md | Decisão | 5 retentativas e depois DLQ | TRANSCRICAO | [09:17] Larissa |
| PRD-DEC-04 | docs/PRD.md | Decisão | Assinatura com secret por webhook e rotação de 24h | TRANSCRICAO | [09:22] Sofia |
| PRD-DEC-05 | docs/PRD.md | Decisão | Entrega at-least-once | TRANSCRICAO | [09:26] Larissa |
| PRD-DEC-06 | docs/PRD.md | Decisão | Módulo com os padrões atuais | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-07 | docs/PRD.md | Decisão | Notificação enxuta como snapshot | TRANSCRICAO | [09:52] Bruno |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança (≥ 2 dias úteis) | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação no portal do desenvolvedor | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Confirmação de prazo com os clientes | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-04 | docs/PRD.md | Dependência | Sessão de revisão de design com Bruno e Diego | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-05 | docs/PRD.md | Dependência | Cliente: endpoint https, verificação e deduplicação | TRANSCRICAO | [09:25] Diego |
| PRD-DEP-06 | docs/PRD.md | Dependência | MySQL e stack atuais | TRANSCRICAO | [09:07] Diego |
| PRD-RISK-01 | docs/PRD.md | Risco | Atraso no prazo e churn da Atlas | TRANSCRICAO | [09:00] Marcos |
| PRD-RISK-02 | docs/PRD.md | Risco | Cliente não deduplica | TRANSCRICAO | [09:25] Sofia |
| PRD-RISK-03 | docs/PRD.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-04 | docs/PRD.md | Risco | Rajadas sobrecarregam o cliente | TRANSCRICAO | [09:38] Diego |
| PRD-RISK-05 | docs/PRD.md | Risco | Cliente fora por mais de ~15h | TRANSCRICAO | [09:17] Marcos |
| PRD-RISK-06 | docs/PRD.md | Risco | Notificações fora de ordem | TRANSCRICAO | [09:13] Larissa |
| PRD-RISK-07 | docs/PRD.md | Risco | Processador parado atrasa notificações | TRANSCRICAO | [09:11] Diego |
| PRD-AC-01 | docs/PRD.md | Critério de aceite | SHIPPED assinado gera notificação assinada em < 10s | TRANSCRICAO | [09:02] Marcos |
| PRD-AC-02 | docs/PRD.md | Critério de aceite | Status não assinado não gera notificação | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-03 | docs/PRD.md | Critério de aceite | URL http recusada | TRANSCRICAO | [09:23] Sofia |
| PRD-AC-04 | docs/PRD.md | Critério de aceite | Retentativas 1m..12h e depois fila de não entregues | TRANSCRICAO | [09:17] Larissa |
| PRD-AC-05 | docs/PRD.md | Critério de aceite | Replay só por admin e registrado | TRANSCRICAO | [09:36] Larissa |
| PRD-AC-06 | docs/PRD.md | Critério de aceite | Rotação com 24h de convivência | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-07 | docs/PRD.md | Critério de aceite | Histórico com até 100 entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-08 | docs/PRD.md | Critério de aceite | Falha ao registrar notificação desfaz a mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-AC-09 | docs/PRD.md | Critério de aceite | Endpoint > 10s conta como falha, sem afetar pedidos | TRANSCRICAO | [09:42] Diego |
| PRD-AC-10 | docs/PRD.md | Critério de aceite | Restart da API ou do processador não perde notificação | TRANSCRICAO | [09:11] Diego |
| PRD-AC-11 | docs/PRD.md | Critério de aceite | Reentrega mantém o mesmo identificador de evento | TRANSCRICAO | [09:25] Diego |
| PRD-TST-01 | docs/PRD.md | Teste | Testes unitários com Vitest | CODIGO | vitest.config.ts |
| PRD-TST-02 | docs/PRD.md | Teste | Integração de API com Supertest e MySQL | CODIGO | tests/orders.test.ts |
| PRD-TST-03 | docs/PRD.md | Teste | Teste de atomicidade com rollback forçado | TRANSCRICAO | [09:40] Bruno |
| PRD-TST-04 | docs/PRD.md | Teste | Ponta a ponta com endpoint simulado (sucesso, timeout, DLQ) | TRANSCRICAO | [09:46] Larissa |
| PRD-TST-05 | docs/PRD.md | Teste | Resiliência: restart do processador e da API | TRANSCRICAO | [09:11] Diego |
| PRD-TST-06 | docs/PRD.md | Teste | Revisão de segurança (HMAC e geração de secret) | TRANSCRICAO | [09:46] Sofia |
| PRD-TST-07 | docs/PRD.md | Teste | Medição de p95 < 10s e zero perda | TRANSCRICAO | [09:02] Marcos |
| PRD-TST-08 | docs/PRD.md | Teste | Regressão da suíte existente | CODIGO | tests/auth.test.ts |
| RFC-CTX-01 | docs/RFC.md | Restrição | Mudança de status não pode depender do cliente | TRANSCRICAO | [09:04] Bruno |
| RFC-CTX-02 | docs/RFC.md | Restrição | Não pode perder evento | TRANSCRICAO | [09:40] Bruno |
| RFC-PROP-01 | docs/RFC.md | Decisão | Gravação na outbox na transação, com filtro e snapshot | TRANSCRICAO | [09:06] Diego |
| RFC-PROP-02 | docs/RFC.md | Decisão | Worker com entry-point próprio e polling de 2s | TRANSCRICAO | [09:11] Larissa |
| RFC-PROP-03 | docs/RFC.md | Decisão | Entrega HTTP com timeout de 10s, headers e assinatura | TRANSCRICAO | [09:44] Diego |
| RFC-PROP-04 | docs/RFC.md | Decisão | Retry e DLQ com replay ADMIN | TRANSCRICAO | [09:18] Diego |
| RFC-PROP-05 | docs/RFC.md | Decisão | API de configuração, rotação e histórico | TRANSCRICAO | [09:33] Bruno |
| RFC-PROP-06 | docs/RFC.md | Decisão | Módulo em src/modules/webhooks com WEBHOOK_* | TRANSCRICAO | [09:27] Bruno |
| RFC-PROP-07 | docs/RFC.md | Decisão | Evento único `order.status_changed`, sem itens | TRANSCRICAO | [09:43] Diego |
| RFC-PROP-08 | docs/RFC.md | Restrição | https, HMAC do corpo, secret por endpoint, grace de 24h | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-09 | docs/RFC.md | Trade-off | At-least-once; ordem só por pedido e com single-worker | TRANSCRICAO | [09:13] Larissa |
| RFC-PLAN-01 | docs/RFC.md | Estimativa | 3 sprints divididas por bloco | TRANSCRICAO | [09:46] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo síncrono em changeStatus | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams (overengineering) | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger no MySQL (sem LISTEN/NOTIFY) | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Retry indefinido ou só 3 tentativas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | `failed` na própria outbox em vez de DLQ | TRANSCRICAO | [09:18] Diego |
| RFC-ALT-06 | docs/RFC.md | Alternativa descartada | Secret global | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-07 | docs/RFC.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-08 | docs/RFC.md | Alternativa descartada | Renderizar o payload no envio | TRANSCRICAO | [09:52] Larissa |
| RFC-Q-01 | docs/RFC.md | Questão em aberto | Rate limiting de envio | TRANSCRICAO | [09:39] Diego |
| RFC-Q-02 | docs/RFC.md | Questão em aberto | E-mail ao cliente em falhas | TRANSCRICAO | [09:37] Marcos |
| RFC-Q-03 | docs/RFC.md | Questão em aberto | Escala para múltiplos workers | TRANSCRICAO | [09:13] Diego |
| RFC-Q-04 | docs/RFC.md | Questão em aberto | Endurecer autorização do CRUD | TRANSCRICAO | [09:37] Sofia |
| RFC-Q-05 | docs/RFC.md | Questão em aberto | Arquivamento de eventos entregues | TRANSCRICAO | [09:08] Diego |
| RFC-L-01 | docs/RFC.md | Lacuna | "5 tentativas" vs. 5 intervalos e ~15h | TRANSCRICAO | [09:17] Diego |
| RFC-L-02 | docs/RFC.md | Lacuna | Assinatura durante o grace period | TRANSCRICAO | [09:21] Sofia |
| RFC-L-03 | docs/RFC.md | Lacuna | X-Timestamp fora da assinatura | TRANSCRICAO | [09:44] Diego |
| RFC-L-04 | docs/RFC.md | Lacuna | Ordem sob retry com single-worker | TRANSCRICAO | [09:12] Diego |
| RFC-L-05 | docs/RFC.md | Lacuna | Como o ADMIN obtém o id da DLQ | TRANSCRICAO | [09:18] Diego |
| RFC-L-06 | docs/RFC.md | Lacuna | Armazenamento da secret em repouso | TRANSCRICAO | [09:46] Sofia |
| RFC-L-07 | docs/RFC.md | Lacuna | Evento na criação do pedido não discutido; v1 publica só em changeStatus | TRANSCRICAO | [09:40] Bruno |
| RFC-RISK-01 | docs/RFC.md | Risco | Atraso no prazo de fim de novembro | TRANSCRICAO | [09:45] Marcos |
| RFC-RISK-02 | docs/RFC.md | Risco | Worker parado acumula eventos | TRANSCRICAO | [09:11] Diego |
| RFC-RISK-03 | docs/RFC.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| RFC-RISK-04 | docs/RFC.md | Risco | Cliente sem deduplicação | TRANSCRICAO | [09:26] Marcos |
| RFC-RISK-05 | docs/RFC.md | Risco | Rajada de eventos no cliente | TRANSCRICAO | [09:38] Diego |
| ADR-001 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Decisão | Outbox transacional no MySQL existente | TRANSCRICAO | [09:08] Larissa |
| ADR-001-ALT-1 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Alternativa descartada | Disparo síncrono | TRANSCRICAO | [09:06] Diego |
| ADR-001-ALT-2 | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Alternativa descartada | Redis Streams | TRANSCRICAO | [09:07] Larissa |
| ADR-001-REF | docs/adrs/ADR-001-outbox-transacional-no-mysql.md | Contexto de código | Transação atual de changeStatus (orders, history, estoque) | CODIGO | src/modules/orders/order.service.ts |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Worker separado, polling de 2s, single-worker | TRANSCRICAO | [09:10] Larissa |
| ADR-002-ALT-1 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Trigger no banco | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-2 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Consumidor dentro do processo da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-REF | docs/adrs/ADR-002-worker-separado-em-polling.md | Contexto de código | Entry-point da API como modelo do worker | CODIGO | src/server.ts |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Decisão | 5 retentativas 1m/5m/30m/2h/12h e DLQ em tabela separada | TRANSCRICAO | [09:17] Larissa |
| ADR-003-ALT-1 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Alternativa descartada | Retry indefinido | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-2 | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Alternativa descartada | 3 tentativas | TRANSCRICAO | [09:16] Bruno |
| ADR-003-REF | docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md | Contexto de código | requireRole reaproveitado no replay | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-004 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 do corpo, secret por endpoint, grace de 24h | TRANSCRICAO | [09:22] Sofia |
| ADR-004-ALT-1 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Alternativa descartada | Secret global | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-2 | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Alternativa descartada | Só TLS, sem assinatura (não atende autenticidade) | TRANSCRICAO | [09:19] Sofia |
| ADR-004-TLS | docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md | Restrição | https obrigatório validado no Zod | TRANSCRICAO | [09:23] Sofia |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | At-least-once com X-Event-Id (UUID) | TRANSCRICAO | [09:26] Larissa |
| ADR-005-ALT-1 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-ALT-2 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Alternativa descartada | At-most-once (incompatível com o retry decidido) | TRANSCRICAO | [09:17] Larissa |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Decisão | Reuso máximo: módulo, AppError, WEBHOOK_*, Pino, error middleware | TRANSCRICAO | [09:30] Larissa |
| ADR-006-ALT-1 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa descartada | Stack própria ("não vamos botar nada novo") | TRANSCRICAO | [09:29] Bruno |
| ADR-006-ALT-2 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Alternativa descartada | Reuso parcial com códigos de erro ad hoc | TRANSCRICAO | [09:29] Larissa |
| ADR-006-REF1 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto de código | Hierarquia AppError e códigos semânticos | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-REF2 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto de código | Middleware de erro trata AppError, Zod e Prisma | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-REF3 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Contexto de código | Logger Pino único | CODIGO | src/shared/logger/index.ts |
| ADR-006-REF4 | docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md | Trade-off | validate() converte ZodError em VALIDATION_ERROR | CODIGO | src/middlewares/validate.middleware.ts |
| ADR-007 | docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md | Decisão | Filtro na inserção e payload snapshot enxuto | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-1 | docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md | Alternativa descartada | Filtrar no envio | TRANSCRICAO | [09:34] Bruno |
| ADR-007-ALT-2 | docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md | Alternativa descartada | Guardar só order_id e renderizar no envio | TRANSCRICAO | [09:51] Bruno |
| FDD-OBJ-01 | docs/FDD.md | Requisito Não Funcional | p95 < 10s; detecção ≤ 2s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBJ-02 | docs/FDD.md | Requisito Não Funcional | Zero evento perdido (atomicidade) | TRANSCRICAO | [09:40] Bruno |
| FDD-OBJ-03 | docs/FDD.md | Requisito Não Funcional | Disponibilidade do cliente sem impacto no PATCH de status | TRANSCRICAO | [09:04] Bruno |
| FDD-OBJ-04 | docs/FDD.md | Requisito Não Funcional | At-least-once deduplicável | TRANSCRICAO | [09:26] Larissa |
| FDD-OBJ-05 | docs/FDD.md | Requisito Não Funcional | Tolerância de ~14h36min antes da DLQ | TRANSCRICAO | [09:17] Diego |
| FDD-OBJ-06 | docs/FDD.md | Restrição | Nenhuma dependência ou infraestrutura nova | TRANSCRICAO | [09:29] Bruno |
| FDD-EXC-01 | docs/FDD.md | Fora de escopo | Webhooks inbound | TRANSCRICAO | [09:02] Marcos |
| FDD-EXC-02 | docs/FDD.md | Fora de escopo | E-mail em falhas | TRANSCRICAO | [09:37] Larissa |
| FDD-EXC-03 | docs/FDD.md | Fora de escopo | Rate limiting | TRANSCRICAO | [09:39] Larissa |
| FDD-EXC-04 | docs/FDD.md | Fora de escopo | Dashboard | TRANSCRICAO | [09:40] Larissa |
| FDD-EXC-05 | docs/FDD.md | Fora de escopo | Arquivamento | TRANSCRICAO | [09:08] Diego |
| FDD-EXC-06 | docs/FDD.md | Fora de escopo | Múltiplos workers e ordem global | TRANSCRICAO | [09:13] Diego |
| FDD-EXC-07 | docs/FDD.md | Fora de escopo | Exactly-once | TRANSCRICAO | [09:25] Diego |
| FDD-EXC-08 | docs/FDD.md | Fora de escopo | Eventos de criação e exclusão de pedido (interpretação, RFC-L-07) | TRANSCRICAO | [09:40] Bruno |
| FDD-EXC-09 | docs/FDD.md | Fora de escopo | Itens no payload | TRANSCRICAO | [09:43] Diego |
| FDD-DADOS-01 | docs/FDD.md | Modelo de dados | Tabela webhooks: url, secret, customer_id, ativo (+ secret anterior) | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-02 | docs/FDD.md | Modelo de dados | webhook_outbox com 4 estados e índices status/created_at | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-03 | docs/FDD.md | Proposta de implementação (derivada) | Uma linha de outbox por webhook interessado; id = event_id | TRANSCRICAO | [09:25] Diego |
| FDD-DADOS-04 | docs/FDD.md | Modelo de dados | webhook_deliveries alimenta o histórico | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-05 | docs/FDD.md | Modelo de dados | webhook_dead_letter com payload, motivo, timestamp e auditoria de replay; sem FK (proposta) | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-06 | docs/FDD.md | Proposta de implementação (derivada) | events em coluna JSON, filtro em memória | TRANSCRICAO | [09:33] Marcos |
| FDD-DADOS-07 | docs/FDD.md | Integração | Relação Customer → Webhook em cascata | CODIGO | prisma/schema.prisma |
| FDD-DADOS-UUID | docs/FDD.md | Restrição | Todas as tabelas novas com UUID | TRANSCRICAO | [09:51] Larissa |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Inserção na outbox dentro de changeStatus, com rollback em falha | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-01a | docs/FDD.md | Decisão | Função publishWebhookEvent(tx, order, from, to) recebendo o tx | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-01b | docs/FDD.md | Proposta de implementação (derivada) | Limite de 64KB checado no worker, não na transação | TRANSCRICAO | [09:23] Sofia |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Worker: busca PENDING, marca PROCESSING, envia, registra | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-02a | docs/FDD.md | Proposta de implementação (derivada) | Concorrência por grupo de orderId, com limite | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-02b | docs/FDD.md | Proposta de implementação (derivada) | BATCH_SIZE inicial de 50 ("batch pequeno") | TRANSCRICAO | [09:08] Diego |
| FDD-FLUXO-02c | docs/FDD.md | Proposta de implementação (derivada) | Pausa: eventos de webhook inativo ficam PENDING até a reativação | TRANSCRICAO | [09:21] Bruno |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Tabela de envios 1+5 com backoff | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | DLQ e replay que recoloca como pendente | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-04a | docs/FDD.md | Proposta de implementação (derivada) | Replay zera tentativas e responde 202/409/404 | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | Rotação: secret anterior válida por 24h | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-06 | docs/FDD.md | Fluxo | Recuperação PROCESSING→PENDING no boot; graceful shutdown | CODIGO | src/server.ts |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST /api/v1/webhooks | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET /api/v1/customers/:customerId/webhooks (customer_id no path) | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | PATCH /api/v1/webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | DELETE /api/v1/webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04a | docs/FDD.md | Proposta de implementação (derivada) | DELETE preserva os registros da DLQ como evidência | TRANSCRICAO | [09:18] Diego |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | POST /api/v1/webhooks/:id/rotate-secret | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | GET /api/v1/webhooks/:id/deliveries: até 100 eventos com suas tentativas | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | POST /api/v1/admin/webhooks/dead-letter/:id/replay (ADMIN) | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Chamada outbound: payload e headers X-Event-Id, X-Webhook-Id, X-Timestamp, X-Signature | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-08a | docs/FDD.md | Proposta de implementação (derivada) | Formato `sha256=<hex>` e redirects não seguidos | TRANSCRICAO | [09:20] Sofia |
| FDD-CONTRATO-CFG | docs/FDD.md | Restrição | customerId no body/query, nunca do JWT | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRATO-ROLE | docs/FDD.md | Decisão | CRUD aberto a qualquer role autenticada | TRANSCRICAO | [09:37] Sofia |
| FDD-ERR-01 | docs/FDD.md | Erro | WEBHOOK_NOT_FOUND (404) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-02 | docs/FDD.md | Erro | WEBHOOK_CUSTOMER_NOT_FOUND (404) | TRANSCRICAO | [09:29] Larissa |
| FDD-ERR-03 | docs/FDD.md | Erro | WEBHOOK_INVALID_URL em details de VALIDATION_ERROR | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-04 | docs/FDD.md | Erro | WEBHOOK_DEAD_LETTER_NOT_FOUND (404) | TRANSCRICAO | [09:29] Larissa |
| FDD-ERR-05 | docs/FDD.md | Erro | WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED (409) | TRANSCRICAO | [09:29] Larissa |
| FDD-ERR-06 | docs/FDD.md | Erro | WEBHOOK_PUBLISH_FAILED (500) com rollback | TRANSCRICAO | [09:40] Bruno |
| FDD-ERR-07 | docs/FDD.md | Erro | WEBHOOK_DELIVERY_TIMEOUT (retentável) | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-08 | docs/FDD.md | Erro | WEBHOOK_DELIVERY_HTTP_ERROR (retentável) | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-09 | docs/FDD.md | Erro | WEBHOOK_DELIVERY_NETWORK_ERROR (retentável) | TRANSCRICAO | [09:14] Larissa |
| FDD-ERR-10 | docs/FDD.md | Erro | WEBHOOK_MAX_ATTEMPTS_EXCEEDED → DLQ | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-11 | docs/FDD.md | Erro | WEBHOOK_PAYLOAD_TOO_LARGE → DLQ sem retry | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-12 | docs/FDD.md | Erro | WEBHOOK_SECRET_REQUIRED (invariante do assinador) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-REUSE | docs/FDD.md | Integração | Reuso de UNAUTHORIZED, FORBIDDEN e VALIDATION_ERROR | CODIGO | src/shared/errors/http-errors.ts |
| FDD-RES-01 | docs/FDD.md | Resiliência | HTTP fora da transação; outbox dentro | TRANSCRICAO | [09:06] Diego |
| FDD-RES-02 | docs/FDD.md | Resiliência | Timeout de 10s | TRANSCRICAO | [09:42] Diego |
| FDD-RES-03 | docs/FDD.md | Resiliência | Backoff exponencial 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Diego |
| FDD-RES-04 | docs/FDD.md | Resiliência | DLQ em tabela separada | TRANSCRICAO | [09:18] Diego |
| FDD-RES-05 | docs/FDD.md | Resiliência | Fallback = DLQ + replay (e-mail adiado) | TRANSCRICAO | [09:37] Larissa |
| FDD-RES-06 | docs/FDD.md | Proposta de implementação (derivada) | Isolamento entre clientes por concorrência limitada e polling que não espera envios em voo | TRANSCRICAO | [09:42] Diego |
| FDD-RES-07 | docs/FDD.md | Resiliência | Recuperação de crash coberta por at-least-once | TRANSCRICAO | [09:24] Diego |
| FDD-RES-08 | docs/FDD.md | Resiliência | Ciclo de vida independente da API | TRANSCRICAO | [09:11] Diego |
| FDD-RES-09 | docs/FDD.md | Resiliência | Worker parado acumula sem perda | TRANSCRICAO | [09:06] Diego |
| FDD-RES-10 | docs/FDD.md | Resiliência | Falha fechada: sem evento, sem mudança de status | TRANSCRICAO | [09:40] Bruno |
| FDD-RES-11 | docs/FDD.md | Proposta de implementação (derivada) | Worker único imposto por lock nomeado GET_LOCK no MySQL | TRANSCRICAO | [09:12] Diego |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Log webhook_event_enqueued | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | Log webhook_delivery_succeeded | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | Log webhook_delivery_failed | TRANSCRICAO | [09:42] Diego |
| FDD-OBS-04 | docs/FDD.md | Observabilidade | Log webhook_event_dead_lettered | TRANSCRICAO | [09:18] Diego |
| FDD-OBS-05 | docs/FDD.md | Observabilidade | Log de auditoria do replay com userId | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-06 | docs/FDD.md | Observabilidade | Log de rotação sem expor a secret | TRANSCRICAO | [09:22] Diego |
| FDD-OBS-07 | docs/FDD.md | Observabilidade | Logs de ciclo de vida do worker (padrão de server_started) | CODIGO | src/server.ts |
| FDD-OBS-REDACT | docs/FDD.md | Proposta de implementação (derivada) | Incluir secret e previousSecret em redactPaths | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-08 | docs/FDD.md | Métrica | Latência até a 1ª tentativa, p95 < 10s (p95 é critério do FDD) | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-09 | docs/FDD.md | Métrica | Idade do pendente mais antigo (alerta > 10s é proposta) | TRANSCRICAO | [09:10] Larissa |
| FDD-OBS-10 | docs/FDD.md | Métrica | Backlog da outbox | TRANSCRICAO | [09:07] Bruno |
| FDD-OBS-11 | docs/FDD.md | Métrica | Taxa de sucesso por webhook (insumo para rate limiting e e-mail) | TRANSCRICAO | [09:39] Diego |
| FDD-OBS-12 | docs/FDD.md | Métrica | Entradas na DLQ e replays (alerta é proposta) | TRANSCRICAO | [09:18] Diego |
| FDD-OBS-13 | docs/FDD.md | Métrica | Duração das chamadas HTTP | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-TRACE | docs/FDD.md | Observabilidade | Tracing lógico por order_id → event_id → X-Event-Id | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-OBS-NOLIB | docs/FDD.md | Restrição | Sem biblioteca de métricas ou tracing no projeto | CODIGO | package.json |
| FDD-INT-01 | docs/FDD.md | Integração | changeStatus estendido com publishWebhookEvent no tx | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Erros WEBHOOK_* estendem AppError e subclasses | CODIGO | src/shared/errors/app-error.ts |
| FDD-INT-03 | docs/FDD.md | Integração | errorMiddleware sem alteração | CODIGO | src/middlewares/error.middleware.ts |
| FDD-INT-04 | docs/FDD.md | Integração | validate() e o efeito em WEBHOOK_INVALID_URL | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-INT-05 | docs/FDD.md | Integração | authenticate e requireRole('ADMIN') | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-06 | docs/FDD.md | Integração | Registro dos routers /webhooks e /admin/webhooks | CODIGO | src/routes/index.ts |
| FDD-INT-07 | docs/FDD.md | Integração | buildControllers instancia o módulo | CODIGO | src/app.ts |
| FDD-INT-08 | docs/FDD.md | Integração | src/worker.ts (novo) espelha bootstrap e shutdown | CODIGO | src/server.ts |
| FDD-INT-09 | docs/FDD.md | Integração | PrismaClient próprio por processo | CODIGO | src/config/database.ts |
| FDD-INT-10 | docs/FDD.md | Integração | Worker valida o mesmo env (JWT_SECRET obrigatório) | CODIGO | src/config/env.ts |
| FDD-INT-11 | docs/FDD.md | Integração | Mesmo logger Pino com redaction ampliada | CODIGO | src/shared/logger/index.ts |
| FDD-INT-12 | docs/FDD.md | Integração | paginated() na listagem | CODIGO | src/shared/http/response.ts |
| FDD-INT-13 | docs/FDD.md | Integração | Status válidos e PENDING que nunca dispara | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-14 | docs/FDD.md | Integração | Padrão de schemas Zod | CODIGO | src/modules/orders/order.schemas.ts |
| FDD-INT-15 | docs/FDD.md | Integração | uuid v4 para event_id | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-INT-16 | docs/FDD.md | Integração | Novos modelos e migração | CODIGO | prisma/schema.prisma |
| FDD-INT-17 | docs/FDD.md | Integração | Scripts worker e worker:dev | CODIGO | package.json |
| FDD-INT-18 | docs/FDD.md | Integração | Limpeza das tabelas novas e factory de webhook nos testes | CODIGO | tests/setup.ts |
| FDD-DEP-01 | docs/FDD.md | Dependência | Node ≥ 20: fetch, AbortSignal.timeout, crypto | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Dependência | MySQL 8.0 e Prisma 5.22; migração aditiva | CODIGO | docker-compose.yml |
| FDD-DEP-03 | docs/FDD.md | Dependência | uuid, zod, pino, express já presentes | CODIGO | package.json |
| FDD-DEP-04 | docs/FDD.md | Compatibilidade | PATCH de status ganha modo de falha 500 com rollback | TRANSCRICAO | [09:40] Bruno |
| FDD-DEP-05 | docs/FDD.md | Compatibilidade | Outbox sem FK para orders; DELETE /orders inalterado | CODIGO | src/modules/orders/order.service.ts |
| FDD-DEP-06 | docs/FDD.md | Proposta de implementação (derivada) | Ordem de deploy: migração → API → worker | TRANSCRICAO | [09:11] Diego |
| FDD-DEP-07 | docs/FDD.md | Dependência | Revisão da Sofia e portal do PM | TRANSCRICAO | [09:46] Sofia |
| FDD-AC-01 | docs/FDD.md | Critério de aceite | N webhooks interessados = N linhas na mesma transação | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-02 | docs/FDD.md | Critério de aceite | Sem interessado = nenhuma linha | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-03 | docs/FDD.md | Critério de aceite | Falha na outbox mantém o status anterior | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-04 | docs/FDD.md | Critério de aceite | Payload exato, sem items, como snapshot | TRANSCRICAO | [09:43] Diego |
| FDD-AC-05 | docs/FDD.md | Critério de aceite | 1ª tentativa ≤ 2s + processamento; p95 < 10s | TRANSCRICAO | [09:09] Diego |
| FDD-AC-06 | docs/FDD.md | Critério de aceite | Headers presentes e assinatura válida | TRANSCRICAO | [09:44] Diego |
| FDD-AC-07 | docs/FDD.md | Critério de aceite | > 10s = timeout e retry em +1 min | TRANSCRICAO | [09:42] Diego |
| FDD-AC-08 | docs/FDD.md | Critério de aceite | 6º envio falho = FAILED + DLQ | TRANSCRICAO | [09:17] Larissa |
| FDD-AC-09 | docs/FDD.md | Critério de aceite | Replay: ADMIN 202, OPERATOR 403, repetido 409, auditado | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-10 | docs/FDD.md | Critério de aceite | http:// devolve 400 com WEBHOOK_INVALID_URL | TRANSCRICAO | [09:23] Sofia |
| FDD-AC-11 | docs/FDD.md | Critério de aceite | Grace de 24h verificável com as duas secrets | TRANSCRICAO | [09:21] Sofia |
| FDD-AC-12 | docs/FDD.md | Critério de aceite | Histórico com até 100 eventos e suas tentativas | TRANSCRICAO | [09:34] Marcos |
| FDD-AC-13 | docs/FDD.md | Critério de aceite | Kill/restart do worker sem perda | TRANSCRICAO | [09:11] Diego |
| FDD-AC-14 | docs/FDD.md | Critério de aceite | Secret ausente de logs e listagens | TRANSCRICAO | [09:22] Diego |
| FDD-AC-15 | docs/FDD.md | Critério de aceite | > 64KB direto para a DLQ, sem HTTP | TRANSCRICAO | [09:24] Diego |
| FDD-AC-16 | docs/FDD.md | Critério de aceite | Suíte existente continua passando | CODIGO | tests/orders.test.ts |
| FDD-AC-17 | docs/FDD.md | Critério de aceite | Segundo worker não obtém o lock e encerra | TRANSCRICAO | [09:12] Diego |
| FDD-AC-18 | docs/FDD.md | Critério de aceite | Webhook desativado não recebe envios; eventos entregues após reativação | TRANSCRICAO | [09:21] Bruno |
| FDD-RISK-01 | docs/FDD.md | Risco | Cliente lento bloqueia os demais | TRANSCRICAO | [09:04] Bruno |
| FDD-RISK-02 | docs/FDD.md | Risco | Ordem quebrada sob retry | TRANSCRICAO | [09:12] Diego |
| FDD-RISK-03 | docs/FDD.md | Risco | Transação de changeStatus mais pesada | TRANSCRICAO | [09:04] Bruno |
| FDD-RISK-04 | docs/FDD.md | Risco | Crescimento da outbox | TRANSCRICAO | [09:07] Bruno |
| FDD-RISK-05 | docs/FDD.md | Risco | Duplicatas | TRANSCRICAO | [09:24] Diego |
| FDD-RISK-06 | docs/FDD.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| FDD-RISK-07 | docs/FDD.md | Risco | URL interna (SSRF), a avaliar na revisão de segurança | TRANSCRICAO | [09:19] Sofia |
| FDD-RISK-08 | docs/FDD.md | Risco | Worker parado sem detecção | TRANSCRICAO | [09:11] Diego |
| FDD-RISK-09 | docs/FDD.md | Risco | Dois workers simultâneos em deploy com sobreposição | TRANSCRICAO | [09:13] Diego |
| FDD-RISK-10 | docs/FDD.md | Risco | Backlog de webhook pausado entregue em rajada | TRANSCRICAO | [09:38] Diego |
