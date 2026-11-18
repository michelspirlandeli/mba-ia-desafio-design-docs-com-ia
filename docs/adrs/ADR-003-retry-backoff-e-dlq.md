# ADR-003 — Política de retry com backoff e Dead Letter Queue em tabela separada

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 2026-11-16 (decisão tomada na reunião técnica; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:17]–[09:19], [09:36]) |
| **Decisores** | Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos), Sofia (Segurança, parte de autorização do replay) |
| **Consultados** | Marcos (PM) |
| **Relacionados** | [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) |

## Contexto

O endpoint do cliente pode estar offline, lento ou respondendo com erro ([09:14] Larissa). Já houve cliente com **indisponibilidade de duas horas** em manutenção planejada ([09:16] Diego). Precisamos tentar de novo sem ficar com eventos pendurados para sempre e sem desistir cedo demais.

## Problema

Quantas vezes tentar, com que intervalos e o que fazer com o evento quando as tentativas se esgotarem?

## Decisão

### Retry

- **5 tentativas de retry após o envio inicial**, com intervalos de **1 min → 5 min → 30 min → 2 h → 12 h** ([09:17] Diego, decidido em [09:17] Larissa e confirmado no resumo de [09:48] Larissa: "Retry com backoff exponencial 1m/5m/30m/2h/12h, total 5 tentativas").

#### Como interpretar "5 tentativas"

Na reunião, "tentativa" foi usada no sentido de **retentativa**: as 5 tentativas são as de retry, uma para cada intervalo. O envio inicial não entra nessa conta. Três falas deixam isso claro:

| Fala | Se "5 tentativas" incluísse o envio inicial | Como retentativas |
|---|---|---|
| [09:15] Diego: "Cinco já dá pra cobrir uma janela de até 12 ou 24 horas" | 4 intervalos, 1m+5m+30m+2h ≈ 2h36 ✗ | 5 intervalos ≈ 14h36 ✓ |
| [09:16] Diego, sobre a proposta de 3: "retentaria três vezes em 30 minutos" | 2 retentativas, 1m+5m = 6 min ✗ | 1m+5m+30m = 36 min ≈ 30 min ✓ |
| [09:17] Diego: "Total de quase 15 horas entre primeira falha e última tentativa" | O intervalo de 12h nunca seria usado ✗ | 1+5+30+120+720 = 876 min ≈ 14h36 ✓ |

A frase de [09:48] coloca lado a lado os **5 intervalos de retry** e o "total 5 tentativas", o que confirma a correspondência de uma tentativa por intervalo.

**Consequência operacional:** cada evento recebe até **6 requisições HTTP** (1 envio inicial + 5 tentativas de retry).

Essa interpretação foi questionada na revisão humana, a partir da fala de [09:48], e mantida com base nas evidências acima.
- A reunião chamou a política de "backoff exponencial" ([09:15] Diego). Na prática é uma **tabela fixa de intervalos crescentes**, implementada como constante e não como fórmula `base * 2^n`.
- **Conta como falha retentável:** resposta não-2xx, erro de rede/TLS e **timeout de 10s** ([09:42] Diego: "Cliente lento que não responde em 10s a gente trata como falha e marca pra retry").

### Dead Letter Queue

- Depois da última retentativa sem sucesso, o evento vira **falha permanente** e é movido para a DLQ ([09:15] Diego).
- A DLQ é uma **tabela separada `webhook_dead_letter`**, com payload, motivo da falha e timestamp ([09:18] Diego; confirmado no resumo de [09:48] Larissa). A linha original da outbox fica com `status = FAILED`.
- **Exceção:** evento com payload acima de 64KB vai **direto para a DLQ**, sem retry, com motivo `WEBHOOK_PAYLOAD_TOO_LARGE`. O limite vem da reunião ([09:23] Sofia, [09:24] Diego, [09:24] Larissa). A escolha de validar no worker, e não na inserção, foi validada na fase de discovery. Validar na inserção faria a mudança de status do pedido falhar por causa de um problema de webhook.

### Reprocessamento (replay)

- Manual, via **`POST /admin/webhooks/dead-letter/:id/replay`** ([09:18] Diego, [09:35] Diego), que recoloca o evento na outbox como pendente ([09:18] Diego).
- Exige **role `ADMIN`**, reaproveitando `requireRole` de [src/middlewares/auth.middleware.ts](../../src/middlewares/auth.middleware.ts) ([09:36] Sofia, [09:36] Larissa).
- Registra **quem** fez o replay, para auditoria ([09:36] Sofia).
- O replay preserva o **mesmo `event_id`** e zera o contador de tentativas ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).

## Justificativa

- Cinco retentativas cobrem uma janela de ~15h, suficiente para manutenções planejadas longas ([09:16] Diego). Marcos considera aceitável: "Se um cliente meu cair por 15 horas, ele já tá com problema sério dele" ([09:17] Marcos).
- A tabela separada mantém a outbox enxuta para o polling e preserva a evidência para debug e reprocessamento ([09:18] Diego).

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Retry indefinido com backoff** | [09:15] Diego | Evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego). |
| **3 tentativas** | [09:16] Bruno | Cobre só ~30 minutos e mataria eventos durante indisponibilidades comuns de manhã ou de manutenção ([09:16] Diego). |
| **DLQ como status `failed` na própria outbox** | [09:17] Larissa | Polui a leitura da outbox principal. A tabela separada é "mais limpa" e serve de evidência ([09:18] Diego). |

## Consequências

### Positivas

- Tolerância a indisponibilidades de até ~14h36 sem intervenção humana.
- Nenhum evento é descartado silenciosamente: ou é entregue, ou fica auditável na DLQ.
- O replay é controlado e auditado (ADMIN + log de quem executou).

### Negativas

- **O evento pode chegar até ~15h depois.** Para o cliente, um evento tardio pode já não refletir o estado atual do pedido, o que é aceitável por ser um snapshot ([ADR-007](ADR-007-payload-snapshot-na-insercao.md)).
- **A ordem por pedido se perde quando há retry** ([ADR-002](ADR-002-worker-separado-em-polling.md)).
- **Nenhum aviso proativo ao cliente** quando o webhook dele falha: e-mail de alerta foi **adiado para a próxima fase** ([09:37] Larissa).
- **O replay é só manual e item a item.** Não há listagem nem replay em massa da DLQ nesta fase (não discutido).
- Os intervalos ficam fixos em código/configuração. Mudar a política exige deploy.

### Trade-off explícito

Priorizamos **não perder eventos e não sobrecarregar o worker com retries infinitos**, aceitando **entregas muito tardias** e **intervenção manual** para os casos esgotados.

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:14]–[09:19], [09:23]–[09:24], [09:35]–[09:36], [09:42], [09:48]
- [src/middlewares/auth.middleware.ts](../../src/middlewares/auth.middleware.ts): `requireRole`
- [FDD §7.3 Retry e backoff](../FDD.md#73-retry-e-backoff) e [FDD §7.4 DLQ e replay](../FDD.md#74-dlq-e-replay)
