# ADR-007 — Payload do evento renderizado como snapshot na inserção da outbox

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 2026-11-16 (decisão tomada no fim da reunião; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:52]) |
| **Decisores** | Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos) |
| **Consultados** | Marcos (PM, formato do payload em [09:43]) |
| **Relacionados** | [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) |

## Contexto

Entre a inserção na outbox e o envio efetivo podem passar de ~2 segundos (polling, [ADR-002](ADR-002-worker-separado-em-polling.md)) a ~15 horas (retries, [ADR-003](ADR-003-retry-backoff-e-dlq.md)), ou mais em caso de replay. Nesse intervalo o pedido pode mudar de novo. O modelo `Order` ([prisma/schema.prisma](../../prisma/schema.prisma)) tem `orderNumber`, `customerId`, `status` e `totalCents`, além de itens em tabela separada.

## Problema

O evento guarda o payload já pronto ou só uma referência (`order_id`) para montar o payload na hora do envio? E o que entra no payload?

## Decisão

1. **Snapshot na inserção:** o payload JSON é montado dentro de `publishWebhookEvent`, na mesma transação de `changeStatus`, e gravado na coluna `payload` da outbox ([09:52] Larissa; [09:52] Diego: "snapshot na inserção"; [09:52] Bruno: "Decidido").
2. **Campos do payload** ([09:43] Diego):

   | Campo | Origem do valor |
   |---|---|
   | `event_id` | UUID da linha da outbox ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)) |
   | `event_type` | constante `"order.status_changed"` |
   | `timestamp` | instante da mudança de status, em ISO 8601 |
   | `order_id` | `Order.id` |
   | `order_number` | `Order.orderNumber` (formato `ORD-000123`, gerado em `reserveOrderNumber`) |
   | `from_status` | status antes da transição |
   | `to_status` | status depois da transição |
   | `customer_id` | `Order.customerId` |
   | `total_cents` | `Order.totalCents` |

3. **Sem itens do pedido.** Se o cliente quiser detalhes, consulta `GET /orders/:id` ([09:43] Diego; [09:44] Bruno: "mantém payload enxuto").
4. Convenção `snake_case` no payload externo, conforme ditado na reunião, diferente do `camelCase` das respostas da API interna. A diferença é intencional e fica documentada no FDD.

## Justificativa

- O evento reflete o estado **no momento da mudança**. Renderizar na hora do envio criaria "caso esquisito": um evento `PAID → PROCESSING` entregue com dados de um pedido já `CANCELLED` ([09:52] Larissa).
- O worker não precisa ler `orders`, o que reduz acoplamento e consultas no processamento.
- Retry e replay enviam **exatamente os mesmos bytes**, o que é coerente com a deduplicação por `event_id` e com a assinatura HMAC.

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Guardar só `order_id` e renderizar no envio** | [09:51] Bruno | O payload deixaria de representar o momento do evento ([09:52] Larissa). |
| **Incluir os itens do pedido no payload** | [09:43] Diego | Infla o payload sem necessidade. O cliente busca os detalhes sob demanda ([09:43] Diego). |

## Consequências

### Positivas

- Fidelidade temporal do evento.
- Retries idempotentes do ponto de vista do conteúdo.
- O payload enxuto fica muito abaixo do limite de 64KB ([09:24] Diego: "Nenhum evento nosso vai chegar perto disso").

### Negativas

- **Armazenamento:** cada linha da outbox guarda o JSON completo, multiplicado pelo número de endpoints interessados ([ADR-001](ADR-001-outbox-no-mysql.md)).
- **Dados possivelmente defasados na entrega tardia.** É intencional, mas o cliente precisa entender que o payload é histórico (documentação no portal, [09:26] Marcos).
- **Evolução do formato:** como o payload fica congelado, mudar o schema do evento afeta eventos já gravados. **Versionamento do payload não foi discutido** e fica registrado como questão em aberto no [RFC](../RFC.md#7-questões-em-aberto).
- Os dados de pedido ficam replicados na outbox. A retenção depende do arquivamento, que está fora de escopo ([09:08] Diego).

### Trade-off explícito

Mais armazenamento e payload potencialmente defasado em troca de **fidelidade ao momento do evento e de reenvios determinísticos**.

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:43]–[09:44], [09:51]–[09:52]
- [src/modules/orders/order.service.ts](../../src/modules/orders/order.service.ts): `changeStatus`, `reserveOrderNumber`
- [prisma/schema.prisma](../../prisma/schema.prisma): modelo `Order`
- [FDD §8.1 Contrato de saída](../FDD.md#81-contrato-de-saída-requisição-enviada-ao-cliente)
