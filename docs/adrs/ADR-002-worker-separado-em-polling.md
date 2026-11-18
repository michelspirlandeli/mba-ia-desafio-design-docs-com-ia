# ADR-002 — Worker em processo separado, lendo a outbox por polling

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 2026-11-16 (decisão tomada na reunião técnica; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:10]–[09:13]) |
| **Decisores** | Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos) |
| **Consultados** | Marcos (PM) |
| **Relacionados** | [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-e-dlq.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) |

## Contexto

Com a outbox definida ([ADR-001](ADR-001-outbox-no-mysql.md)), falta definir **quem lê** a tabela e **como** descobre que há eventos novos. Restrições:

- Os clientes consideram "tempo real" qualquer coisa abaixo de **10 segundos** ([09:02] Marcos).
- O MySQL não tem mecanismo nativo de notificação para processos externos ([09:09] Diego).
- A aplicação hoje tem um único entry point, [src/server.ts](../../src/server.ts), que sobe a API Express e faz graceful shutdown.
- O Prisma Client é criado por [src/config/database.ts](../../src/config/database.ts) (`createPrismaClient()` + singleton `prisma`).

## Problema

Como consumir a outbox de forma confiável, dentro da latência aceita e sem ficar sujeito aos reinícios da API?

## Decisão

1. **Processo separado:** o worker roda como processo Node próprio, **não** dentro da instância da API ([09:11] Diego).
   - Novo entry point `src/worker.ts`, ao lado de `src/server.ts`, com script `npm run worker` ([09:11] Larissa).
   - Mesmo banco e mesma stack, mas processo diferente ([09:11] Diego).
   - **PrismaClient próprio**, com a mesma `DATABASE_URL`, porque o client é por processo ([09:30] Bruno). Obtido via `createPrismaClient()`.
   - A lógica de processamento fica dentro do módulo, em `src/modules/webhooks/webhook.processor.ts` ([09:28] Bruno ofereceu `webhook.worker.ts` ou `webhook.processor.ts`; escolhemos `processor` para não confundir com o entry point `src/worker.ts`).
2. **Polling em loop a cada 2 segundos:** busca os eventos pendentes mais antigos em **lote pequeno**, processa e marca ([09:08] Diego, [09:09] Diego). Decisão registrada em [09:10] Larissa: "Worker em polling, 2s".
3. **Instância única (single-worker):** os eventos são processados em ordem de `created_at`. A ordenação é garantida **apenas por `order_id`** e **apenas enquanto houver um único worker**. Isso fica registrado como **limitação conhecida** ([09:12] Diego, [09:13] Larissa).

## Justificativa

- Polling de 2s cabe com folga no limite de 10s ([09:09] Diego; [09:10] Marcos: "2 segundos serve").
- Processo separado: um reinício ou deploy da API não interrompe as entregas, e uma falha no worker não derruba a API ([09:11] Diego).
- Single-worker dispensa locks distribuídos e particionamento agora ("problema do futuro", [09:13] Diego).

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Trigger de banco / LISTEN-NOTIFY para reagir a inserts** | [09:09] Bruno | O MySQL não tem LISTEN/NOTIFY como o Postgres. Uma trigger só executa SQL e não notifica processo externo. Avisar o worker exigiria improviso (arquivo, endpoint) ([09:09] Diego). |
| **Worker dentro do processo da API** | [09:11] Diego | Se a API reinicia, o worker cai junto ([09:11] Diego). |
| **Múltiplos workers em paralelo** (particionados por `order_id` ou com lock pessimista) | [09:13] Diego | Perde a garantia de ordem e exige coordenação. Fica adiado como evolução ([09:13] Diego). |

## Consequências

### Positivas

- Latência de detecção de **no máximo ~2s**, dentro do requisito de 10s ([09:10] Larissa: "A latência mínima vai ser 2 segundos no pior caso. Aceitamos.").
- Isolamento operacional entre API e entrega de webhooks.
- Reuso total da stack: TypeScript, Prisma, Pino, `env.ts` ([ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)).

### Negativas

- **Consulta constante ao MySQL** (uma a cada 2s, mesmo sem eventos). O custo é baixo graças ao índice em `status` ([ADR-001](ADR-001-outbox-no-mysql.md)).
- **Mais um processo para implantar e monitorar.** Se ele parar, os eventos se acumulam em `PENDING` sem erro visível na API. Por isso a observabilidade do worker é obrigatória ([FDD §12](../FDD.md#12-observabilidade)).
- **Head-of-line blocking:** com um único worker processando em sequência, um cliente lento (até 10s de timeout, [09:42] Diego) atrasa os eventos dos demais clientes.
- **A ordem por `order_id` não sobrevive a retries.** Se o evento `PAID` de um pedido falhar e entrar em backoff, o `PROCESSING` seguinte do mesmo pedido pode ser entregue antes. A garantia afirmada em [09:12] vale no caminho feliz. Os clientes não pediram ordem global ([09:14] Marcos), e o payload traz `from_status`, `to_status` e `timestamp` para que o cliente reordene se precisar. Ponto registrado como questão em aberto no [RFC](../RFC.md#7-questões-em-aberto).
- **Nada impede, por construção, que alguém suba dois workers.** O FDD mitiga com claim condicional (`UPDATE ... WHERE status = 'PENDING'`), mas a ordem deixa de ser garantida.

### Trade-off explícito

Trocamos **throughput e escalabilidade horizontal** por **simplicidade operacional e ordenação por pedido no caminho feliz**. Escalar exigirá revisitar esta decisão ([09:13] Diego).

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:08]–[09:14], [09:28], [09:30]
- [src/server.ts](../../src/server.ts): modelo de bootstrap e shutdown a replicar em `src/worker.ts`
- [src/config/database.ts](../../src/config/database.ts): `createPrismaClient()`
- [FDD §7.2 Processamento pelo worker](../FDD.md#72-processamento-pelo-worker)
