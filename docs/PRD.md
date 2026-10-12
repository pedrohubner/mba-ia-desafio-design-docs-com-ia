# PRD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
|---|---|
| **Produto** | Order Management System (OMS) |
| **Feature** | Webhooks de notificação de mudança de status de pedidos |
| **Autor** | Pedro Hubner |
| **PM responsável** | Marcos |
| **Status** | Aprovado em reunião técnica; detalhamento em revisão |
| **Data** | 2026-10-12 |
| **Cronograma** | 3 sprints, com entrega para a Atlas até o fim de novembro (ver [RFC §3.4](RFC.md#34-plano-de-entrega)) |
| **Documentos relacionados** | [RFC](RFC.md) · [FDD](FDD.md) · [ADRs](adrs/README.md) · [Tracker](TRACKER.md) |

---

## 1. Resumo e contexto da feature

O OMS passa a **avisar os clientes B2B, de forma ativa e em segundos, sempre que o status de um pedido deles muda**. O cliente cadastra uma URL, escolhe os status que quer acompanhar e recebe uma notificação HTTP assinada a cada mudança. Notificações que falham são retentadas automaticamente por até ~15 horas. As que não forem entregues ficam guardadas para reprocessamento.

Hoje o OMS não tem nenhum mecanismo de notificação externa. Quem quer saber de uma mudança precisa perguntar.

## 2. Problema e motivação

- **Três clientes B2B pediram formalmente a funcionalidade:** Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos).
- **Hoje eles consultam a listagem de pedidos de tempos em tempos** para descobrir mudanças. A integração fica lenta e cara para eles ([09:00] Marcos).
- **Há risco comercial concreto:** a Atlas indicou que pode migrar para um concorrente se a funcionalidade não sair até o fim do trimestre ([09:00] Marcos).
- **A expectativa dos clientes é simples:** saber de cada mudança em menos de 10 segundos, sem precisar atualizar manualmente ([09:02] Marcos).

## 3. Público-alvo e cenários de uso

### 3.1 Público-alvo

| Persona | Descrição | Origem |
|---|---|---|
| **Desenvolvedor de integração do cliente B2B** | Implementa o endpoint que recebe as notificações, valida a assinatura e deduplica eventos | [09:00] Marcos, [09:25] Diego |
| **Usuário do OMS que representa o cliente** | Cadastra e mantém os webhooks pela API, autenticado no OMS | [09:32] Marcos |
| **Administrador do OMS** | Reprocessa notificações que esgotaram as tentativas | [09:36] Sofia |
| **PM** | Documenta o contrato de integração no portal do desenvolvedor | [09:26] Marcos, [09:40] Marcos |

### 3.2 Cenários de uso

| ID | Cenário |
|---|---|
| PRD-CEN-01 | Um cliente cadastra um webhook para ser avisada só quando o pedido vira **SHIPPED** ou **DELIVERED** ([09:33] Marcos). Segundos depois de o operador marcar um pedido como enviado, o sistema do cliente recebe a notificação. |
| PRD-CEN-02 | O endpoint de um cliente fica 2 horas fora do ar em manutenção planejada ([09:16] Diego). As notificações do período são retentadas e entregues quando o endpoint volta, sem ninguém intervir. |
| PRD-CEN-03 | Um cliente percebe que vazou a secret em log ([09:22] Diego), pede uma nova pela API e tem 24 horas para atualizar os sistemas sem perder notificações ([09:21] Sofia). |
| PRD-CEN-04 | Um cliente diz que não recebeu uma notificação. Ele consulta o histórico das últimas entregas e vê sucesso/falha, conteúdo, resposta e tempo de resposta ([09:34] Marcos). |
| PRD-CEN-05 | Um cliente ficou mais de 15 horas fora do ar. As notificações esgotaram as tentativas e foram guardadas. Depois do contato, um administrador as reprocessa ([09:18] Diego). |
| PRD-CEN-06 | Um cliente recebe a mesma notificação duas vezes e descarta a repetida pelo identificador único do evento ([09:25] Diego). |

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta | Origem |
|---|---|---|---|---|
| PRD-MET-01 | Notificação "em tempo real" na percepção do cliente | Tempo entre a mudança de status e a 1ª tentativa de entrega (p95) | **< 10 segundos** | [09:02] Marcos |
| PRD-MET-02 | Detecção rápida da mudança | Intervalo máximo até o evento ser percebido para envio | **≤ 2 segundos** | [09:10] Larissa |
| PRD-MET-03 | Confiabilidade | Mudanças de status com webhook interessado que não geraram notificação | **0 (zero)** | [09:40] Bruno |
| PRD-MET-04 | Resistência a indisponibilidade do cliente | Janela de indisponibilidade coberta sem intervenção humana | **≈ 14h36min** (interpretação em confirmação: RFC-L-01) | [09:17] Diego/Marcos |
| PRD-MET-05 | Retenção dos clientes solicitantes | Clientes solicitantes integrados e recebendo notificações | **3 de 3** (Atlas, MaxDistribuição, Nova Cargo), Atlas até **fim de novembro** | [09:00] Marcos, [09:45] Marcos |
| PRD-MET-06 | Previsibilidade de entrega | Duração do desenvolvimento, incluindo revisão de segurança | **3 sprints** | [09:47] Larissa |
| PRD-MET-07 | Menos polling | Volume de consultas à listagem de pedidos pelos 3 clientes | Acompanhar a tendência de queda (**sem meta numérica definida na reunião**) | [09:00] Marcos |
| PRD-MET-08 | Insumo para a próxima fase | Taxa de falha e volume na fila de não entregues, por cliente | Acompanhar para decidir e-mail e rate limiting | [09:37] Larissa, [09:39] Larissa |

> **Notas de medição.** A reunião fixou só o teto ("abaixo de 10 segundos"). Medir esse teto em **p95** é uma escolha deste PRD para tornar a meta verificável. A meta "3 de 3" deriva dos três clientes que fizeram o pedido formal.

## 5. Escopo

### 5.1 Incluso

- Cadastro, listagem, edição, ativação/desativação e remoção de webhooks por cliente.
- Escolha, por webhook, dos status de pedido que geram notificação.
- Notificação a cada mudança de status de pedido, com conteúdo enxuto.
- Assinatura de cada notificação e troca de secret com período de convivência.
- Retentativas automáticas e fila de notificações não entregues, com reprocessamento por administrador.
- Histórico das últimas entregas por webhook.

### 5.2 Fora de escopo

| ID | Item | Situação | Origem |
|---|---|---|---|
| PRD-OOS-01 | **Aviso ao cliente por e-mail** quando o webhook dele falha repetidamente | **Adiado** para a próxima fase, depois de medir o impacto | [09:37] Larissa |
| PRD-OOS-02 | **Painel visual (dashboard)** para o cliente acompanhar os webhooks | **Descartado** desta feature: é projeto separado do time de frontend. Na v1 só há API. | [09:40] Larissa |
| PRD-OOS-03 | **Limitação de taxa (rate limiting)** de notificações por cliente | **Adiado**: observar o comportamento real e decidir depois | [09:39] Larissa |
| PRD-OOS-04 | **Webhooks de entrada** (cliente enviando dados ao OMS) | **Descartado**: os clientes só querem receber | [09:02] Marcos |
| PRD-OOS-05 | **Arquivamento** de notificações antigas já entregues | **Fora do escopo** desta feature | [09:08] Diego |
| PRD-OOS-06 | **Garantia de ordem global** e processamento paralelo em vários workers | **Adiado** ("problema do futuro"). Os clientes não pediram ordem global. | [09:13] Diego, [09:14] Marcos |
| PRD-OOS-07 | **Entrega exatamente uma vez** | **Descartado**: complexidade desproporcional | [09:25] Diego |
| PRD-OOS-08 | **Restrição de perfil no cadastro de webhooks** | **Adiado**: qualquer usuário autenticado pode cadastrar por enquanto | [09:37] Sofia |
| PRD-OOS-09 | **Itens do pedido no conteúdo da notificação** | **Descartado**: o cliente consulta o pedido se precisar | [09:43] Diego |

## 6. Requisitos funcionais

| ID | Requisito | Origem |
|---|---|---|
| PRD-FR-01 | O usuário autenticado deve poder **cadastrar um webhook** para um cliente, informando URL e lista de status de interesse. O cliente é identificado explicitamente na requisição, não pelo login. | [09:31] Marcos, [09:32] Larissa |
| PRD-FR-02 | A plataforma deve **gerar a secret** do webhook e devolvê-la ao usuário na criação | [09:31] Marcos |
| PRD-FR-03 | O usuário deve poder **listar os webhooks de um cliente** | [09:33] Bruno |
| PRD-FR-04 | O usuário deve poder **editar** um webhook (URL, status de interesse, ativo/inativo) | [09:33] Bruno, [09:21] Bruno/Sofia |
| PRD-FR-05 | O usuário deve poder **remover** um webhook | [09:33] Bruno |
| PRD-FR-06 | Cada webhook deve receber **só os status que escolheu**. Se nenhum webhook do cliente quer aquele status, nada é gerado. | [09:33] Marcos, [09:34] Bruno |
| PRD-FR-07 | Toda **mudança de status de pedido** deve gerar uma notificação do tipo `order.status_changed` para cada webhook interessado, com identificador do evento, momento, pedido, número do pedido, status anterior e novo, cliente e valor total | [09:43] Diego |
| PRD-FR-08 | Cada notificação deve ser **assinada** para o cliente comprovar origem e integridade | [09:19] Sofia, [09:20] Sofia |
| PRD-FR-09 | O usuário deve poder **solicitar uma nova secret**. A anterior continua válida por **24 horas**. | [09:21] Sofia |
| PRD-FR-10 | Notificações não entregues devem ser **retentadas automaticamente**, com espera crescente de 1 min, 5 min, 30 min, 2 h e 12 h ("5 tentativas"; a leitura como 5 retentativas depois do envio inicial está em confirmação na RFC-L-01) | [09:17] Larissa |
| PRD-FR-11 | Esgotadas as retentativas, a notificação deve ir para uma **fila de não entregues** com conteúdo, motivo e momento da falha | [09:18] Diego |
| PRD-FR-12 | Um **administrador** deve poder **reprocessar** uma notificação da fila de não entregues, e a ação deve ser registrada com o autor | [09:18] Diego, [09:36] Sofia |
| PRD-FR-13 | O usuário deve poder **consultar o histórico das últimas 100 entregas** de um webhook, com sucesso/falha, conteúdo, resposta e tempo de resposta | [09:34] Marcos |
| PRD-FR-14 | Cada notificação deve informar o **identificador único do evento** e o **identificador do webhook** que a originou | [09:25] Diego, [09:44] Sofia |

## 7. Requisitos não funcionais

| ID | Requisito | Origem |
|---|---|---|
| PRD-NFR-01 | **Latência:** notificação em menos de 10s após a mudança. A latência mínima de até 2s por polling é aceita. | [09:02] Marcos, [09:10] Larissa |
| PRD-NFR-02 | **Atomicidade:** se o status mudou, a notificação será gerada; se a mudança foi desfeita, não será | [09:06] Diego, [09:40] Bruno |
| PRD-NFR-03 | **Isolamento:** lentidão ou indisponibilidade de um cliente não pode afetar mudanças de status de pedidos | [09:04] Bruno |
| PRD-NFR-04 | **Garantia de entrega at-least-once:** o cliente pode receber duplicatas e deve deduplicar pelo identificador do evento | [09:24] Diego, [09:26] Larissa |
| PRD-NFR-05 | **Ordem:** só por pedido, no fluxo normal e com um único processador. Sob retentativa pode haver inversão (RFC-L-04). Não há ordem global. | [09:13] Larissa |
| PRD-NFR-06 | **Segurança de transporte:** só URLs `https`. URLs `http` são recusadas no cadastro. | [09:23] Sofia |
| PRD-NFR-07 | **Segurança de credencial:** secret única por webhook, nunca global | [09:21] Sofia |
| PRD-NFR-08 | **Tamanho:** notificações acima de **64KB** não são enviadas e geram erro (não truncam) | [09:24] Larissa |
| PRD-NFR-09 | **Tempo de resposta do cliente:** o cliente tem até **10s** para responder. Depois disso, conta como falha e entra em retentativa. | [09:42] Diego |
| PRD-NFR-10 | **Disponibilidade do processamento:** o envio roda separado da API e não para quando a API reinicia | [09:11] Diego |
| PRD-NFR-11 | **Sem infraestrutura nova:** a solução usa o banco e a stack atuais | [09:07] Diego, [09:30] Larissa |
| PRD-NFR-12 | **Auditoria:** reprocessamentos ficam registrados com quem os executou | [09:36] Sofia |
| PRD-NFR-13 | **Conteúdo enxuto e imutável:** a notificação reflete o pedido no momento da mudança e não leva itens | [09:43] Diego, [09:52] Larissa |

## 8. Decisões e trade-offs principais

| ID | Decisão | Trade-off aceito | ADR |
|---|---|---|---|
| PRD-DEC-01 | Registrar a notificação junto com a mudança de status, no banco atual, e enviar depois | Alguns segundos de latência em troca de nunca perder notificação e de não subir infraestrutura | [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md) |
| PRD-DEC-02 | Processador separado, verificando novidades a cada 2s | Latência mínima de 2s e um processo a mais em troca de simplicidade | [ADR-002](adrs/ADR-002-worker-separado-em-polling.md) |
| PRD-DEC-03 | Retentativas por até ~15h (RFC-L-01) e depois fila de não entregues | Clientes fora do ar por mais tempo dependem de reprocessamento manual | [ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) |
| PRD-DEC-04 | Assinatura com secret por webhook e troca com 24h de convivência | O cliente precisa implementar a verificação | [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) |
| PRD-DEC-05 | Entrega ao menos uma vez | O cliente precisa deduplicar | [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| PRD-DEC-06 | Construir como mais um módulo do OMS, com os padrões atuais | Fica restrito às ferramentas atuais (por exemplo, sem biblioteca de métricas) | [ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) |
| PRD-DEC-07 | Notificação enxuta, congelada no momento da mudança | O cliente consulta o pedido para ver itens e detalhes | [ADR-007](adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md) |

## 9. Dependências

| ID | Dependência | Responsável | Origem |
|---|---|---|---|
| PRD-DEP-01 | **Revisão de segurança** (assinatura e geração de secret), com pelo menos 2 dias úteis antes do deploy | Sofia | [09:46] Sofia |
| PRD-DEP-02 | **Documentação no portal do desenvolvedor**: como integrar, verificar a assinatura e deduplicar | Marcos | [09:26] Marcos, [09:40] Marcos |
| PRD-DEP-03 | **Confirmação de prazo** com os clientes | Marcos | [09:47] Marcos |
| PRD-DEP-04 | **Sessão de revisão do design** antes de começar a codar | Larissa, Bruno, Diego | [09:50] Larissa |
| PRD-DEP-05 | **Do lado do cliente**: endpoint `https`, verificação de assinatura e deduplicação por evento | Clientes B2B | [09:23] Sofia, [09:25] Diego |
| PRD-DEP-06 | **Banco MySQL e stack atuais** do OMS | Time de Pedidos/Plataforma | [09:07] Diego |

## 10. Riscos e mitigação

> A classificação de probabilidade e impacto é avaliação do autor. Os riscos e as mitigações vêm das falas citadas.

| ID | Risco | Probabilidade | Impacto | Mitigação | Origem |
|---|---|---|---|---|---|
| PRD-RISK-01 | **Atraso em relação ao fim de novembro**, levando à perda da Atlas | Média | Alto | Escopo enxuto (e-mail, dashboard e rate limiting fora); estimativa de 3 sprints já com revisão de segurança; prazo confirmado com o cliente | [09:00] Marcos, [09:46] Larissa |
| PRD-RISK-02 | **Cliente não deduplica** e processa a mesma notificação duas vezes | Média | Médio | Contrato at-least-once em destaque no portal; identificador único em toda notificação | [09:25] Sofia, [09:26] Marcos |
| PRD-RISK-03 | **Vazamento da secret** de um cliente | Média | Alto | Secret por webhook (dano isolado); troca com 24h de convivência; revisão de segurança | [09:21] Sofia, [09:22] Diego |
| PRD-RISK-04 | **Rajadas de notificações** sobrecarregam o endpoint do cliente | Média | Médio | Monitorar a taxa de falha por cliente (PRD-MET-08); decidir sobre rate limiting na próxima fase | [09:38] Diego, [09:39] Larissa |
| PRD-RISK-05 | **Cliente fora do ar por mais de ~15h** e as notificações param de ser tentadas | Baixa | Médio | Fila de não entregues com reprocessamento por administrador; aviso por e-mail avaliado na próxima fase | [09:17] Marcos, [09:37] Larissa |
| PRD-RISK-06 | **Notificações fora de ordem** quando há falha e retentativa | Média | Baixo | Limitação documentada (RFC-L-04); os clientes não exigem ordem global. Proposta: o cliente usa o momento da mudança, presente em toda notificação, para descartar estado antigo. | [09:13] Larissa, [09:14] Marcos |
| PRD-RISK-07 | **Processador parado** atrasa as notificações (sem perder nenhuma) | Baixa | Alto | Processo dedicado ([09:11] Diego). Proposta do FDD: alerta sobre o tempo de espera da notificação mais antiga. | [09:11] Diego |

## 11. Critérios de aceitação

| ID | Critério (Dado / Quando / Então) | Requisitos |
|---|---|---|
| PRD-AC-01 | **Dado** um webhook ativo que assina SHIPPED, **quando** um pedido do cliente muda para SHIPPED, **então** o endpoint recebe uma notificação assinada em menos de 10s (p95). | FR-01, FR-07, FR-08, NFR-01 |
| PRD-AC-02 | **Dado** um webhook que assina só DELIVERED, **quando** o pedido muda para SHIPPED, **então** nenhuma notificação é gerada. | FR-06 |
| PRD-AC-03 | **Dado** um cadastro com URL `http://`, **quando** ele é enviado, **então** é recusado com erro de validação. | NFR-06 |
| PRD-AC-04 | **Dado** um endpoint fora do ar, **quando** a notificação falha, **então** é retentada em 1 min, 5 min, 30 min, 2 h e 12 h e, se todas falharem, vai para a fila de não entregues com o motivo. | FR-10, FR-11 |
| PRD-AC-05 | **Dado** uma notificação na fila de não entregues, **quando** um administrador a reprocessa, **então** ela volta a ser enviada e o registro mostra quem reprocessou. **Quando** um não administrador tenta, **então** recebe acesso negado. | FR-12, NFR-12 |
| PRD-AC-06 | **Dado** um pedido de nova secret, **então** a nova é devolvida e notificações das próximas 24h podem ser validadas com a antiga ou com a nova. Depois disso, só com a nova. O mecanismo de assinatura no período está em definição (RFC-L-02). | FR-09 |
| PRD-AC-07 | **Dado** um webhook com entregas, **quando** o histórico é consultado, **então** aparecem até as 100 mais recentes, com sucesso/falha, conteúdo, resposta e tempo de resposta. | FR-13 |
| PRD-AC-08 | **Dado** uma falha ao registrar a notificação, **quando** o status do pedido é alterado, **então** a alteração é desfeita e nada fica inconsistente. | NFR-02 |
| PRD-AC-09 | **Dado** um endpoint que demora mais de 10s, **então** a tentativa conta como falha e é retentada, sem afetar outras mudanças de status. | NFR-03, NFR-09 |
| PRD-AC-10 | **Dado** que a API reinicia, **então** o envio de notificações continua; e se o processador reinicia, nenhuma notificação é perdida. | NFR-10, NFR-02 |
| PRD-AC-11 | Toda notificação traz o identificador único do evento e o do webhook, e uma reentrega traz o mesmo identificador de evento. | FR-14, NFR-04 |

## 12. Estratégia de testes e validação

| ID | Nível | O que valida | Como |
|---|---|---|---|
| PRD-TST-01 | **Unitário** | Assinatura, cálculo de espera entre tentativas, filtro de status, montagem do conteúdo e limite de 64KB | Vitest, como no projeto (`vitest.config.ts`) |
| PRD-TST-02 | **Integração (API)** | Cadastro, listagem, edição, remoção, troca de secret, histórico, reprocessamento e permissões | Vitest + Supertest + MySQL, no padrão de `tests/orders.test.ts` |
| PRD-TST-03 | **Integração (atomicidade)** | Mudança de status + registro da notificação, incluindo falha forçada com o pedido mantendo o status anterior | Teste com o banco real, como os testes de estoque existentes |
| PRD-TST-04 | **Ponta a ponta** | Processador + endpoint simulado: sucesso, timeout, erro, retentativas, fila de não entregues e reprocessamento | Servidor HTTP local de teste no papel do cliente |
| PRD-TST-05 | **Resiliência** | Reinício do processador no meio do envio; API reiniciando com o processador ativo | Testes de falha induzida |
| PRD-TST-06 | **Segurança** | Assinatura, geração de secret, ausência de secrets em logs e respostas | Revisão dedicada da Sofia, com pelo menos 2 dias úteis ([09:46] Sofia) |
| PRD-TST-07 | **Validação de métricas** | p95 < 10s e zero perda | Medição em ambiente pré-produção a partir do próprio registro de entregas (ver FDD §9) |
| PRD-TST-08 | **Regressão** | A suíte atual de pedidos e autenticação continua passando | `npm test` |
