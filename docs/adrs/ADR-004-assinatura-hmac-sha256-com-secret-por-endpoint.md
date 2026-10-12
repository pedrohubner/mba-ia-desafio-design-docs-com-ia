# ADR-004: Autenticação das entregas com HMAC-SHA256 e secret por endpoint, com rotação

**Status:** Aceito
**Data:** 2026-10-12 (decisão tomada na reunião técnica de quinta-feira, 09:00)
**ADRs relacionados:** [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md)

## Status

Aceito. Decidido por Sofia (Segurança) em [09:22] e confirmado no resumo de [09:48]. A implementação passa por **revisão de segurança dedicada da Sofia, de pelo menos 2 dias úteis, antes do deploy** ([09:46] Sofia).

## Contexto

Os webhooks levam dados de pedidos para endpoints **fora da nossa infraestrutura**. O fluxo é só outbound, do OMS para o cliente ([09:02] Marcos, [09:03] Sofia). O cliente precisa conseguir validar que a requisição veio de nós e que o payload não foi adulterado no caminho ([09:19] Sofia).

Há histórico de vazamento: um cliente já expôs uma secret em log da própria aplicação ([09:22] Diego). Se uma única credencial for comprometida, o dano precisa ficar restrito a um endpoint.

## Direcionadores da Decisão

- Autenticidade e integridade verificáveis pelo cliente ([09:19] Sofia).
- Padrão de mercado com biblioteca disponível em qualquer stack ([09:20] Sofia).
- Contenção de dano em vazamento: "se vaza uma, vaza tudo" ([09:21] Sofia).
- Permitir troca de credencial sem downtime do lado do cliente ([09:21] Sofia).

## Alternativas Consideradas

1. **Secret global da plataforma** assinando todos os envios ([09:21] Sofia, como contraexemplo).
2. **Sem assinatura, confiando só em TLS.** É plausível, mas não atende ao requisito de autenticidade de [09:19].
3. **HMAC-SHA256 do corpo com secret única por endpoint, rotacionável com grace period de 24h** ([09:20]–[09:22] Sofia).

## Decisão

Adotamos a **alternativa 3**:

- Cada envio carrega a assinatura **HMAC-SHA256 calculada sobre o corpo do request**, no header **`X-Signature`** ([09:20] Sofia, [09:22] Sofia).
- **Uma secret por endpoint de webhook**, nunca global ([09:21] Sofia). A configuração guarda URL, secret, `customer_id` e estado ativo ([09:21] Bruno/Sofia).
- A secret é **gerada pela plataforma e devolvida ao cliente na criação** do webhook ([09:31] Marcos).
- **Rotação via API**: o cliente pede uma nova secret, e a antiga **continua válida por 24 horas em paralelo**. Depois disso, deixa de valer ([09:21] Sofia).
- **TLS obrigatório**: URL que não seja `https` é recusada com erro de validação no schema Zod ([09:23] Sofia). Isso é uma validação, não uma decisão arquitetural separada.

## Prós e Contras das Alternativas

### Secret global
- ✅ Uma credencial só para gerenciar.
- ❌ Um vazamento compromete todos os clientes ([09:21] Sofia).
- ❌ Não dá para rotacionar a credencial de um cliente sem afetar os demais.

### Só TLS
- ✅ Nenhum esforço adicional do cliente.
- ❌ O cliente não tem como provar que a chamada partiu do OMS e não de um terceiro que conhece a URL.

### HMAC-SHA256 por endpoint com rotação
- ✅ Padrão de mercado, com biblioteca em qualquer linguagem ([09:20] Sofia).
- ✅ Vazamento isolado num único endpoint, e a rotação resolve sem downtime.
- ❌ Exige que o cliente implemente a verificação.
- ❌ A secret precisa ficar armazenada de forma recuperável, porque HMAC exige a chave original. Isso aumenta a superfície de ataque do banco.

## Consequências

**Positivas:** o cliente valida origem e integridade com uma primitiva conhecida. O comprometimento de uma secret fica restrito a um endpoint e se remedia por rotação, sem janela de indisponibilidade.

**Negativas:** a plataforma passa a guardar segredos de terceiros. O vazamento citado em [09:22] Diego aconteceu no log do próprio cliente, e o FDD propõe evitar o equivalente do nosso lado com redaction da secret no logger. Três detalhes ficaram para a revisão da Sofia: RFC-L-02 (assinatura no grace), RFC-L-03 (`X-Timestamp` fora do conteúdo assinado) e RFC-L-06 (armazenamento em repouso), todos no [RFC](../RFC.md#5-questões-em-aberto).

**Trade-off explícito:** colocamos parte da responsabilidade de segurança no cliente (verificar a assinatura) em troca de um mecanismo padrão, barato e com dano contido.

## Referências

- `src/modules/orders/order.schemas.ts` (padrão de schemas Zod a seguir; a regra `https` entra no novo `webhook.schemas.ts`)
- `src/shared/logger/index.ts` (lista `redactPaths` do Pino, que precisa incluir a secret)
- `src/config/env.ts` (precedente de validação de segredo com tamanho mínimo, `JWT_SECRET`)
- Transcrição: [09:19]–[09:23] Sofia/Bruno/Diego, [09:31] Marcos, [09:46] Sofia
