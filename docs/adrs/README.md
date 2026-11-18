# Architectural Decision Records

Este diretório armazena os ADRs (Architectural Decision Records) do projeto. Cada decisão arquitetural relevante fica em um arquivo próprio, nomeado no formato `ADR-NNN-titulo-em-kebab-case.md` e escrito em variante do formato MADR.

## Índice: Sistema de Webhooks de Notificação de Pedidos

| ADR | Decisão | Status |
|---|---|---|
| [ADR-001](ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL, na mesma transação de `changeStatus` | Aceito |
| [ADR-002](ADR-002-worker-separado-em-polling.md) | Worker em processo separado, polling de 2s, instância única | Aceito |
| [ADR-003](ADR-003-retry-backoff-e-dlq.md) | Retry 1m/5m/30m/2h/12h e DLQ em tabela separada com replay ADMIN | Aceito |
| [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação com grace de 24h | Aceito (detalhes de rotação pendentes) |
| [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com `X-Event-Id` | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões existentes (módulos, AppError, Pino, Zod, `requireRole`) | Aceito |
| [ADR-007](ADR-007-payload-snapshot-na-insercao.md) | Payload renderizado como snapshot na inserção | Aceito |

Contexto geral: [RFC](../RFC.md) · Implementação: [FDD](../FDD.md) · Rastreabilidade: [TRACKER](../TRACKER.md)
