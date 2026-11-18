# ADR-006 — Reuso dos padrões existentes do projeto no módulo de webhooks

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 2026-11-16 (decisão tomada na reunião técnica; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:30]) |
| **Decisores** | Larissa (Tech Lead), Bruno (Pedidos) |
| **Consultados** | Diego (Plataforma), Sofia (Segurança) |
| **Relacionados** | [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-003](ADR-003-retry-backoff-e-dlq.md) |

## Contexto

A codebase tem convenções claras e consistentes ([09:27] Bruno), confirmadas na leitura do código:

| Padrão | Onde está no código |
|---|---|
| Módulo por domínio: `*.routes.ts`, `*.controller.ts`, `*.service.ts`, `*.repository.ts`, `*.schemas.ts` | [src/modules/orders/](../../src/modules/orders/), [src/modules/customers/](../../src/modules/customers/) etc. |
| Classe base de erro `AppError(message, statusCode, errorCode, details)` | [src/shared/errors/app-error.ts](../../src/shared/errors/app-error.ts) |
| Subclasses com código `UPPER_SNAKE` (`INSUFFICIENT_STOCK`, `INVALID_STATUS_TRANSITION`) | [src/shared/errors/http-errors.ts](../../src/shared/errors/http-errors.ts) |
| Tratamento centralizado de `AppError`, `ZodError` e Prisma (`P2002`, `P2025`) | [src/middlewares/error.middleware.ts](../../src/middlewares/error.middleware.ts) |
| Validação Zod de `body`, `query` e `params` | [src/middlewares/validate.middleware.ts](../../src/middlewares/validate.middleware.ts) |
| Autenticação JWT e autorização por papel | [src/middlewares/auth.middleware.ts](../../src/middlewares/auth.middleware.ts) (`authenticate`, `requireRole`) |
| Logger Pino com `redact` | [src/shared/logger/index.ts](../../src/shared/logger/index.ts) |
| Composição manual de dependências e montagem de rotas | [src/app.ts](../../src/app.ts) (`buildControllers`), [src/routes/index.ts](../../src/routes/index.ts) (`buildApiRouter`) |
| Resposta paginada `{ data, pagination }` | [src/shared/http/response.ts](../../src/shared/http/response.ts) |

## Problema

O módulo de webhooks deve introduzir padrões próprios (framework, logger, formato de erro) ou se encaixar no que já existe?

## Decisão

**Reuso máximo do que já existe; webhooks é um módulo igual aos outros** ([09:30] Larissa):

1. **Estrutura:** `src/modules/webhooks/` com `webhook.routes.ts`, `webhook.controller.ts`, `webhook.service.ts`, `webhook.repository.ts`, `webhook.schemas.ts` ([09:27] Bruno), mais `webhook.processor.ts` para a lógica do worker ([09:28] Bruno; ver [ADR-002](ADR-002-worker-separado-em-polling.md)).
2. **Erros:** classes do módulo estendem `AppError`, com códigos sempre prefixados por **`WEBHOOK_`** ([09:28] Bruno, [09:29] Larissa), por exemplo `WEBHOOK_NOT_FOUND`.
3. **Logger:** o Pino que já existe, sem biblioteca nova ([09:29] Bruno).
4. **Middleware de erro:** sem alteração. Ele já trata qualquer `AppError` ([09:29] Bruno).
5. **Validação:** schemas Zod com o `validate()` existente ([09:30] Larissa).
6. **Autorização:** `requireRole('ADMIN')` no replay da DLQ ([09:36] Larissa). As demais rotas usam só `authenticate` ([09:37] Sofia).
7. **Prisma:** mesmo schema e mesma `DATABASE_URL`. O worker cria a sua instância com `createPrismaClient()` ([09:30] Bruno).

### Nuances encontradas na leitura do código (ajustes de implementação)

- `NotFoundError` ([http-errors.ts](../../src/shared/errors/http-errors.ts)) fixa o código `NOT_FOUND` e não aceita outro. `WebhookNotFoundError` precisa estender **`AppError` diretamente** (`super('Webhook not found', 404, 'WEBHOOK_NOT_FOUND')`) para ter o prefixo `WEBHOOK_`. `ConflictError` e `UnprocessableEntityError` aceitam código customizado e podem ser estendidas.
- `validate.middleware.ts` converte **todo** `ZodError` em `ValidationError` com código `VALIDATION_ERROR`. Como a regra de HTTPS fica no schema Zod ([09:23] Sofia), uma URL inválida volta como `400 VALIDATION_ERROR`, com `details[].message` começando por `WEBHOOK_INVALID_URL`. **Não alteramos o middleware**, para preservar o reuso. Ver a matriz de erros no [FDD §9](../FDD.md#9-matriz-de-erros).
- O logger tem `base.service = 'order-management-api'` fixo. O worker usa `logger.child({ component: 'webhook-worker' })` para diferenciar a origem sem alterar a fábrica.

## Justificativa

- Menor curva de aprendizado e revisão mais rápida, num prazo de 3 sprints ([09:46] Larissa).
- Contrato de erro HTTP uniforme para os clientes da API: `{ error: { code, message, details? } }`.
- Evita dependências novas, alinhado com a restrição de time pequeno ([09:07] Diego).

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Introduzir bibliotecas novas** (outro logger, container de DI, cliente HTTP dedicado) | [09:29] Bruno: "Não vamos botar nada novo." | Mais superfície de manutenção sem ganho claro. O Node 20 já tem `https`, `dns`, `net.BlockList` e `crypto`, suficientes para o cliente HTTP com validação de destino e para o HMAC ([FDD §11.1](../FDD.md#111-proteção-contra-ssrf-severidade-crítica)). |
| **Serviço de webhooks separado (microsserviço próprio)** | Plausível, não discutida | Exigiria novo deploy, novo banco ou acesso cruzado, e perderia a atomicidade da outbox na mesma transação ([ADR-001](ADR-001-outbox-no-mysql.md)). |
| **Alterar `validate.middleware.ts` para propagar códigos customizados de Zod** | Plausível | Mudaria o contrato de erro de todos os módulos para atender um caso. O ganho não justifica. |

## Consequências

### Positivas

- Código previsível para quem já conhece o projeto.
- Os erros `WEBHOOK_*` saem pelo envelope padrão sem tocar no middleware.
- Testes seguem o mesmo padrão (Vitest + Supertest em [tests/](../../tests/)).

### Negativas

- **Os limites dos padrões são herdados**: por exemplo, o código `WEBHOOK_INVALID_URL` aparece em `details` e não em `error.code`, por causa do `validate.middleware`.
- `requireRole` só conhece `ADMIN | OPERATOR`. Não existe papel de "usuário de cliente", o que limita o endurecimento futuro do CRUD ([09:37] Sofia).
- O módulo `orders` passa a depender do módulo `webhooks` (`publishWebhookEvent`), criando um acoplamento entre módulos que antes não existia.

### Trade-off explícito

Consistência e velocidade de entrega acima de um design "ideal" isolado para webhooks.

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:27]–[09:30], [09:36]–[09:37]
- Arquivos listados na tabela de Contexto
- [FDD §15 Integração com o sistema existente](../FDD.md#15-integração-com-o-sistema-existente)
