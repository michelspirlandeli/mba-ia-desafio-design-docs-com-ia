# ADR-001 — Padrão Outbox no MySQL para eventos de webhook

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 2026-11-16 (decisão tomada na reunião técnica; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:08]) |
| **Decisores** | Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos) |
| **Consultados** | Marcos (PM), Sofia (Segurança) |
| **Relacionados** | [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-payload-snapshot-na-insercao.md), [RFC](../RFC.md), [FDD](../FDD.md) |

## Contexto

O OMS precisa notificar clientes B2B quando o status de um pedido muda ([09:00] Marcos). A mudança de status acontece em `OrderService.changeStatus` ([src/modules/orders/order.service.ts](../../src/modules/orders/order.service.ts)), dentro de um único `prisma.$transaction` que:

1. atualiza `orders.status`;
2. insere em `order_status_history`;
3. debita ou repõe `products.stockQuantity` conforme `shouldDebitStock` / `shouldReplenishStock` ([src/modules/orders/order.status.ts](../../src/modules/orders/order.status.ts)).

A aplicação não possui nenhum broker, fila ou mecanismo de eventos ([package.json](../../package.json) só tem Express, Prisma, Pino, Zod, JWT, bcrypt e uuid). A infraestrutura é um único MySQL 8 ([docker-compose.yml](../../docker-compose.yml)).

## Problema

Como registrar o evento "status do pedido mudou" de forma que:

- se a transação de mudança de status **commitar**, o evento **obrigatoriamente** exista;
- se a transação sofrer **rollback**, o evento **não** exista;
- a disponibilidade ou lentidão do endpoint do cliente **não** afete a mudança de status ([09:04] Bruno)?

## Decisão

Adotar o **padrão Transactional Outbox no MySQL existente** ([09:06] Diego, fechado em [09:08] Larissa):

- Nova tabela `webhook_outbox` no mesmo banco.
- O evento é inserido **dentro da mesma transação** de `changeStatus`, por meio da função `publishWebhookEvent(tx, order, fromStatus, toStatus)`, que recebe o `Prisma.TransactionClient` corrente ([09:41] Bruno, [09:41] Diego).
- Se a inserção na outbox falhar, a transação inteira sofre rollback: não pode existir mudança de status sem evento ([09:40] Bruno, [09:41] Diego).
- Ciclo de vida da linha: `PENDING → PROCESSING → DELIVERED | FAILED` ([09:08] Diego).
- Índices em `status` e `created_at` ([09:08] Diego).
- **Filtro na inserção:** só são inseridos eventos para webhooks ativos do customer cujo filtro contenha o `to_status`. Se nenhum webhook se interessa, nada é inserido ([09:34] Bruno, [09:34] Diego).
- Identificadores UUID, seguindo o restante do schema ([09:51] Larissa; [prisma/schema.prisma](../../prisma/schema.prisma)).
- **Granularidade (detalhamento de modelagem):** uma linha por par *(evento, endpoint de webhook)*. Assim, retry, backoff e DLQ são independentes por endpoint, e um cliente fora do ar não atrasa o retry de outro.

## Justificativa

- A atomicidade vem de graça da transação MySQL que já existe. Não há janela de inconsistência entre "commitou" e "publicou" ([09:06] Diego: "Não tem inconsistência possível").
- Não exige infraestrutura nova, o que importa para um time pequeno ([09:07] Diego).
- Tira o HTTP para o cliente do caminho crítico da transação. Quem entrega é o worker ([ADR-002](ADR-002-worker-separado-em-polling.md)).

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Chamada HTTP síncrona dentro de `changeStatus`** | [09:03] Larissa, [09:04] Bruno | A transação já é pesada. Um cliente lento seguraria a transação e travaria mudanças de status de outros pedidos. Com o cliente fora do ar, não faz sentido fazer rollback da mudança de status ([09:04] Bruno). "Síncrono está fora de questão" ([09:06] Diego). |
| **Redis Streams / fila externa** | [09:07] Larissa | Exige subir e operar mais infraestrutura. Para um time pequeno, um Redis Cluster seria overengineering ([09:07] Diego). |
| **Publicar após o commit (dual write)** | Plausível, não discutida | Se o processo cair entre o commit e a publicação, o evento se perde. Esse é justamente o caso que o outbox elimina. |

## Consequências

### Positivas

- Consistência forte entre mudança de status e registro do evento.
- Nenhum componente novo de infraestrutura: mesmo MySQL, mesmo Prisma, mesmo backup.
- A falha do cliente fica isolada do fluxo de pedidos.
- O filtro na inserção economiza linhas quando o customer não tem interesse no status ([09:34] Bruno).

### Negativas

- **Custo extra na transação crítica:** um `SELECT` dos webhooks do customer e um `INSERT` em lote por mudança de status. É pequeno, mas está no caminho de `PATCH /orders/:id/status`.
- **Acoplamento:** o módulo `orders` passa a importar uma função do módulo `webhooks`, e uma falha nela derruba a mudança de status (comportamento desejado, mas explícito).
- **Crescimento da tabela:** linhas `DELIVERED` se acumulam. O arquivamento após ~30 dias ficou **fora do escopo** desta feature ([09:08] Diego).
- O MySQL passa a servir também como fila, com polling contínuo ([ADR-002](ADR-002-worker-separado-em-polling.md)).

### Trade-off explícito

Aceitamos **latência de entrega maior** (polling) e **carga adicional no MySQL** em troca de **consistência transacional garantida e zero infraestrutura nova**.

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:03]–[09:08], [09:34], [09:40]–[09:41], [09:51]
- [src/modules/orders/order.service.ts](../../src/modules/orders/order.service.ts): método `changeStatus`
- [prisma/schema.prisma](../../prisma/schema.prisma): modelos `Order` e `OrderStatusHistory`
- [FDD §6 Modelo de dados](../FDD.md#6-modelo-de-dados) e [FDD §7.1 Criação do evento](../FDD.md#71-criação-do-evento-na-outbox)
