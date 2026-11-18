# ADR-005 — Garantia de entrega at-least-once com `X-Event-Id` para deduplicação

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 2026-11-16 (decisão tomada na reunião técnica; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:26]) |
| **Decisores** | Larissa (Tech Lead), Diego (Plataforma) |
| **Consultados** | Sofia (Segurança), Bruno (Pedidos), Marcos (PM) |
| **Relacionados** | [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-e-dlq.md), [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) |

## Contexto

Entre o worker e o endpoint do cliente há uma rede não confiável. Existem cenários em que **o cliente processou o evento, mas o OMS não sabe disso**:

- o cliente processou e respondeu depois do timeout de 10s ([09:42] Diego), e o worker registra falha e retenta;
- a conexão caiu depois que o cliente recebeu, mas antes da resposta chegar;
- o worker caiu entre receber o 2xx e marcar a linha como `DELIVERED`;
- um ADMIN fez replay de um evento da DLQ que o cliente, na verdade, já tinha recebido ([ADR-003](ADR-003-retry-backoff-e-dlq.md)).

## Problema

Qual garantia de entrega oferecer e como o cliente lida com duplicatas?

## Decisão

1. **Garantia at-least-once:** o OMS garante que todo evento registrado na outbox será entregue **pelo menos uma vez** (ou irá para a DLQ). O cliente **pode receber o mesmo evento mais de uma vez** e precisa estar preparado ([09:24] Diego, decidido em [09:26] Larissa).
2. **Header `X-Event-Id`:** UUID gerado **quando o evento entra na outbox**, único por evento ([09:25] Diego). O cliente deduplica por esse valor ([09:25] Diego).
3. **Estabilidade do identificador (detalhamento):** o `X-Event-Id` é o `id` da linha em `webhook_outbox` e se repete, igual, em **todos os retries e em todo replay**. Ele também vai no corpo como `event_id` ([09:43] Diego), coberto pela assinatura HMAC.
4. **Header `X-Webhook-Id`:** id do endpoint cadastrado, para o cliente com vários webhooks saber qual cadastro originou o envio ([09:44] Sofia, [09:45] Diego).
5. **Comunicação ao cliente:** Marcos documenta a semântica at-least-once com destaque no portal do desenvolvedor ([09:26] Marcos).

Como a outbox tem uma linha por par *(evento, endpoint)* ([ADR-001](ADR-001-outbox-no-mysql.md)), cada endpoint recebe um `event_id` próprio para a mesma mudança de status. A deduplicação é sempre por endpoint, então isso não traz ambiguidade para o cliente.

## Justificativa

- É o padrão de mercado: "Stripe faz assim, GitHub faz assim" ([09:25] Diego).
- At-least-once com id de evento "resolve 99% dos casos" a um custo de implementação baixo ([09:25] Diego).

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Exactly-once** | [09:25] Diego | Exigiria coordenação dos dois lados (por exemplo, confirmação idempotente do cliente ou protocolo de duas fases) e "fica muito mais complexo" ([09:25] Diego). |
| **At-most-once (sem retry)** | Plausível, não discutida | Perderia eventos sempre que o cliente estivesse indisponível, o que contradiz a política de retry e DLQ ([ADR-003](ADR-003-retry-backoff-e-dlq.md)). |

## Consequências

### Positivas

- O OMS não depende da idempotência do cliente para ser correto. Basta repetir até o 2xx.
- O retry e o replay ficam simples, sem verificar se o cliente "já recebeu".
- O `event_id` vira chave de correlação ponta a ponta (logs, entregas, DLQ, cliente).

### Negativas

- **A responsabilidade de deduplicar vai para o cliente** ([09:25] Sofia: "Isso joga responsabilidade pro cliente."). Um cliente que não deduplica pode processar duas vezes.
- O cliente precisa guardar os `event_id` já vistos por pelo menos a janela de retry (~15h) e mais a possibilidade de replay manual.
- A garantia vale **apenas a partir do commit**: eventos filtrados na inserção ([ADR-001](ADR-001-outbox-no-mysql.md)) ou de webhooks inativos não são entregues, por definição.

### Trade-off explícito

Simplicidade e robustez do lado do OMS em troca de **um contrato que exige deduplicação no cliente**, mitigado com documentação no portal.

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:24]–[09:26], [09:43]–[09:45]
- [FDD §8.1 Contrato do evento enviado ao cliente](../FDD.md#81-contrato-de-saída-requisição-enviada-ao-cliente)
