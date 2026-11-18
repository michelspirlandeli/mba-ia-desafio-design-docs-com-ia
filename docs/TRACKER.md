# TRACKER — Rastreabilidade da documentação

Cada linha liga um item registrado nos documentos à sua origem:

- `TRANSCRICAO`: fala em [TRANSCRICAO.md](../TRANSCRICAO.md), no formato `[hh:mm] Nome`.
- `CODIGO`: arquivo real do repositório.

**Convenções de Tipo**

- **Decisão, Requisito Funcional, Requisito Não Funcional, Restrição, Trade-off, Fora de escopo, Questão em aberto, Alternativa descartada:** vêm diretamente da fonte indicada.
- **Inferência:** conclusão técnica derivada da fonte (por exemplo, uma limitação que decorre de duas decisões).
- **Proposta:** detalhamento técnico **sem decisão explícita na reunião**, criado para tornar o FDD implementável. A Localização aponta para a fala ou o arquivo que motivou a proposta. Proposta **não é** decisão da reunião.
- **Proposta (revisão humana):** igual a Proposta, mas originada na revisão técnica feita pelo autor depois da primeira versão do pacote (SSRF, criptografia da secret, auditoria, evoluções futuras). A Localização aponta para a fala relacionada mais próxima.

A seção [Resumo de cobertura](#resumo-de-cobertura) no fim do documento traz os números da verificação automatizada.

## PRD — [docs/PRD.md](PRD.md)

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-CTX-01 | docs/PRD.md | Contexto | Atlas, MaxDistribuição e Nova Cargo pedem notificação de mudança de status | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Contexto | Clientes fazem polling em GET /orders; integração lenta e cara | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-03 | docs/PRD.md | Restrição | Atlas ameaça migrar para concorrente se não houver entrega até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-04 | docs/PRD.md | Contexto | Ciclo de vida PENDING→PAID→PROCESSING→SHIPPED→DELIVERED / CANCELLED | CODIGO | src/modules/orders/order.status.ts |
| PRD-PER-01 | docs/PRD.md | Público-alvo | Usuários do OMS representam o cliente e configuram via API com JWT | TRANSCRICAO | [09:32] Marcos |
| PRD-PER-02 | docs/PRD.md | Público-alvo | Administrador (role ADMIN) reprocessa DLQ | TRANSCRICAO | [09:36] Sofia |
| PRD-PER-03 | docs/PRD.md | Público-alvo | PM documenta a integração no portal do desenvolvedor | TRANSCRICAO | [09:40] Marcos |
| PRD-CEN-01 | docs/PRD.md | Cenário | Cliente assina só SHIPPED e DELIVERED | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-02 | docs/PRD.md | Cenário | Cliente com indisponibilidade de 2h em manutenção planejada | TRANSCRICAO | [09:16] Diego |
| PRD-CEN-03 | docs/PRD.md | Cenário | Cliente que vazou secret em log de aplicação | TRANSCRICAO | [09:22] Diego |
| PRD-OBJ-01 | docs/PRD.md | Métrica | Latência p95 < 10s ("tempo real" para o cliente) | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Métrica | 3 de 3 clientes solicitantes com webhook ativo | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-03 | docs/PRD.md | Métrica | Entrega em ≤ 3 sprints com revisão de segurança | TRANSCRICAO | [09:47] Larissa |
| PRD-OBJ-04 | docs/PRD.md | Proposta | ≥ 99% de eventos entregues sem replay manual (meta não definida na reunião) | TRANSCRICAO | [09:18] Diego |
| PRD-OBJ-05 | docs/PRD.md | Proposta | Redução do polling em GET /orders (linha de base a coletar) | TRANSCRICAO | [09:00] Marcos |
| PRD-OOS-01 | docs/PRD.md | Fora de escopo | E-mail de alerta ao cliente adiado para a próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-OOS-02 | docs/PRD.md | Fora de escopo | Dashboard visual: projeto do time de frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-OOS-03 | docs/PRD.md | Fora de escopo | Rate limiting de saída: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-OOS-04 | docs/PRD.md | Fora de escopo | Webhooks de entrada: só saída | TRANSCRICAO | [09:02] Marcos |
| PRD-OOS-05 | docs/PRD.md | Fora de escopo | Garantia de ordem global não exigida | TRANSCRICAO | [09:14] Marcos |
| PRD-OOS-06 | docs/PRD.md | Fora de escopo | Arquivamento de eventos entregues após ~30 dias | TRANSCRICAO | [09:08] Diego |
| PRD-OOS-07 | docs/PRD.md | Fora de escopo | Endurecer permissões do CRUD fica para depois | TRANSCRICAO | [09:37] Sofia |
| PRD-FR-001 | docs/PRD.md | Requisito Funcional | Notificar o cliente quando o status do pedido muda | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-002 | docs/PRD.md | Requisito Funcional | Cadastrar webhook (url, status); secret gerada e devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-003 | docs/PRD.md | Requisito Funcional | Editar webhook (PATCH) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-004 | docs/PRD.md | Requisito Funcional | Remover webhook (DELETE) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-005 | docs/PRD.md | Requisito Funcional | Listar webhooks de um customer (GET) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-006 | docs/PRD.md | Requisito Funcional | Filtro por lista de status por endpoint | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-007 | docs/PRD.md | Requisito Funcional | Rotação de secret via API com grace period de 24h | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-008 | docs/PRD.md | Requisito Funcional | Histórico das últimas 100 entregas com payload, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-009 | docs/PRD.md | Requisito Funcional | Retry automático: 5 tentativas 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-010 | docs/PRD.md | Requisito Funcional | Falha permanente vai para a DLQ com o motivo | TRANSCRICAO | [09:18] Diego |
| PRD-FR-011 | docs/PRD.md | Requisito Funcional | Replay manual da DLQ por ADMIN, registrando o autor | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-012 | docs/PRD.md | Requisito Funcional | Identificador de evento e de webhook em cada mensagem | TRANSCRICAO | [09:44] Sofia |
| PRD-FR-013 | docs/PRD.md | Requisito Funcional | Payload com dados essenciais, sem itens | TRANSCRICAO | [09:43] Diego |
| PRD-FR-014 | docs/PRD.md | Requisito Funcional | Estado ativo/inativo do webhook | TRANSCRICAO | [09:21] Bruno |
| PRD-NFR-001 | docs/PRD.md | Requisito Não Funcional | Latência < 10s | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-002 | docs/PRD.md | Requisito Não Funcional | Status nunca muda sem evento registrado (atomicidade) | TRANSCRICAO | [09:40] Bruno |
| PRD-NFR-003 | docs/PRD.md | Requisito Não Funcional | Cliente lento não trava mudança de status | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-004 | docs/PRD.md | Requisito Não Funcional | HMAC-SHA256 com secret exclusiva por webhook | TRANSCRICAO | [09:22] Sofia |
| PRD-NFR-005 | docs/PRD.md | Requisito Não Funcional | Apenas URLs HTTPS | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-006 | docs/PRD.md | Requisito Não Funcional | Limite de 64KB, erro sem truncar | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-007 | docs/PRD.md | Requisito Não Funcional | Entrega at-least-once; cliente deduplica | TRANSCRICAO | [09:26] Larissa |
| PRD-NFR-008 | docs/PRD.md | Requisito Não Funcional | Timeout de 10s do cliente conta como falha | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-009 | docs/PRD.md | Requisito Não Funcional | Auditoria de reenvio manual | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-010 | docs/PRD.md | Restrição | Sem infraestrutura nova (time pequeno) | TRANSCRICAO | [09:07] Diego |
| PRD-NFR-011 | docs/PRD.md | Restrição | Revisão de segurança de ≥ 2 dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-DEC-01 | docs/PRD.md | Trade-off | Assíncrono via outbox: até ~2s de atraso em troca de robustez | TRANSCRICAO | [09:10] Larissa |
| PRD-DEC-02 | docs/PRD.md | Trade-off | Retry por ~15h aceitável para o negócio | TRANSCRICAO | [09:17] Marcos |
| PRD-DEC-03 | docs/PRD.md | Trade-off | At-least-once joga a deduplicação para o cliente | TRANSCRICAO | [09:25] Sofia |
| PRD-DEC-04 | docs/PRD.md | Trade-off | Payload enxuto; cliente consulta GET /orders/:id para detalhes | TRANSCRICAO | [09:43] Diego |
| PRD-DEP-01 | docs/PRD.md | Dependência | Portal do desenvolvedor documenta a deduplicação | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-02 | docs/PRD.md | Dependência | Marcos confirma o prazo com a Atlas | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Prazo esperado pela Atlas: fim de novembro | TRANSCRICAO | [09:45] Marcos |
| PRD-DEP-04 | docs/PRD.md | Dependência | Novo processo (worker) a implantar | TRANSCRICAO | [09:11] Diego |
| PRD-RSK-01 | docs/PRD.md | Risco | Atraso e churn da Atlas; estimativa de 3 sprints | TRANSCRICAO | [09:46] Larissa |
| PRD-RSK-02 | docs/PRD.md | Risco | Cliente processa duplicata | TRANSCRICAO | [09:25] Diego |
| PRD-RSK-03 | docs/PRD.md | Risco | Vazamento de secret (isolado por endpoint) | TRANSCRICAO | [09:21] Sofia |
| PRD-RSK-04 | docs/PRD.md | Risco | 50 eventos/min sobrecarregando o cliente | TRANSCRICAO | [09:38] Diego |
| PRD-RSK-05 | docs/PRD.md | Risco | Atraso por cliente lento / worker parado | TRANSCRICAO | [09:42] Diego |
| PRD-RSK-06 | docs/PRD.md | Risco | Cliente não é avisado da falha (e-mail adiado) | TRANSCRICAO | [09:37] Marcos |
| PRD-AC-001 | docs/PRD.md | Critério de aceitação | Entrega assinada < 10s para status assinado | TRANSCRICAO | [09:02] Marcos |
| PRD-AC-002 | docs/PRD.md | Critério de aceitação | Status não assinado não gera envio | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-003 | docs/PRD.md | Critério de aceitação | Secret devolvida só na criação/rotação | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-004 | docs/PRD.md | Critério de aceitação | URL http recusada | TRANSCRICAO | [09:23] Sofia |
| PRD-AC-005 | docs/PRD.md | Critério de aceitação | Endpoint fora por 2h recebe na retentativa | TRANSCRICAO | [09:16] Diego |
| PRD-AC-006 | docs/PRD.md | Critério de aceitação | DLQ após ~15h; replay só por ADMIN | TRANSCRICAO | [09:36] Larissa |
| PRD-AC-007 | docs/PRD.md | Critério de aceitação | Secret antiga válida por 24h após rotação | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-008 | docs/PRD.md | Critério de aceitação | Reenvio mantém o mesmo identificador de evento | TRANSCRICAO | [09:25] Diego |
| PRD-AC-009 | docs/PRD.md | Critério de aceitação | Histórico mostra até 100 entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-010 | docs/PRD.md | Critério de aceitação | Cliente fora do ar não afeta a operação de pedidos | TRANSCRICAO | [09:04] Bruno |
| PRD-TST-01 | docs/PRD.md | Estratégia de testes | Revisão de segurança focada em HMAC e geração de secret | TRANSCRICAO | [09:46] Sofia |
| PRD-RSK-07 | docs/PRD.md | Proposta (revisão humana) | Risco de SSRF: URL cadastrada alcança sistemas internos | TRANSCRICAO | [09:19] Sofia |
| PRD-RSK-08 | docs/PRD.md | Risco | Cliente vê ou altera webhooks de outro (CRUD aberto a qualquer autenticado) | TRANSCRICAO | [09:37] Sofia |
| PRD-FUT-01 | docs/PRD.md | Proposta (revisão humana) | Indicador de saúde do endpoint (evolução futura) | TRANSCRICAO | [09:37] Marcos |
| PRD-FUT-02 | docs/PRD.md | Proposta (revisão humana) | Evento de teste para validar a integração (evolução futura) | TRANSCRICAO | [09:40] Marcos |

## RFC — [docs/RFC.md](RFC.md)

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| RFC-META-01 | docs/RFC.md | Metadado | Larissa abre o doc de design e agenda revisão com Bruno e Diego | TRANSCRICAO | [09:50] Larissa |
| RFC-META-02 | docs/RFC.md | Metadado | Revisão obrigatória de segurança pela Sofia | TRANSCRICAO | [09:49] Sofia |
| RFC-META-03 | docs/RFC.md | Metadado | Estimativa de 3 sprints | TRANSCRICAO | [09:47] Larissa |
| RFC-CTX-01 | docs/RFC.md | Contexto | changeStatus atualiza orders, history e estoque numa transação | CODIGO | src/modules/orders/order.service.ts |
| RFC-CTX-02 | docs/RFC.md | Contexto | Transação de mudança de status já é pesada | TRANSCRICAO | [09:04] Bruno |
| RFC-PROP-01 | docs/RFC.md | Decisão | Outbox no MySQL na mesma transação | TRANSCRICAO | [09:08] Larissa |
| RFC-PROP-02 | docs/RFC.md | Decisão | Worker separado com polling de 2s | TRANSCRICAO | [09:10] Larissa |
| RFC-PROP-03 | docs/RFC.md | Decisão | Retry 1m/5m/30m/2h/12h e DLQ separada | TRANSCRICAO | [09:48] Larissa |
| RFC-PROP-04 | docs/RFC.md | Decisão | HMAC-SHA256, secret por endpoint, rotação de 24h | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-05 | docs/RFC.md | Decisão | At-least-once com X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| RFC-PROP-06 | docs/RFC.md | Decisão | Módulo src/modules/webhooks seguindo os padrões | TRANSCRICAO | [09:30] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo HTTP síncrono em changeStatus | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams / fila externa (overengineering) | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger de banco / LISTEN-NOTIFY | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Exactly-once (complexidade) | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Retry indefinido / 3 tentativas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-06 | docs/RFC.md | Alternativa descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| RFC-Q-01 | docs/RFC.md | Questão em aberto | Rate limiting de envio por cliente | TRANSCRICAO | [09:39] Diego |
| RFC-Q-02 | docs/RFC.md | Questão em aberto | customer_id no body ou no path | TRANSCRICAO | [09:32] Larissa |
| RFC-Q-03 | docs/RFC.md | Questão em aberto | Escala horizontal: partição por order_id ou lock pessimista | TRANSCRICAO | [09:13] Diego |
| RFC-Q-04 | docs/RFC.md | Questão em aberto | Endurecer autorização do CRUD | TRANSCRICAO | [09:37] Sofia |
| RFC-Q-04b | docs/RFC.md | Inferência | User não tem vínculo com Customer: qualquer usuário configura qualquer customer | CODIGO | prisma/schema.prisma |
| RFC-Q-05 | docs/RFC.md | Questão em aberto | Com qual secret assinar durante o grace de 24h | TRANSCRICAO | [09:21] Sofia |
| RFC-Q-06 | docs/RFC.md | Inferência | Ordem por pedido não sobrevive a retry (garantia vale no caminho feliz) | TRANSCRICAO | [09:12] Diego |
| RFC-Q-07 | docs/RFC.md | Questão em aberto | Emitir evento na criação do pedido (null → PENDING)? | CODIGO | src/modules/orders/order.service.ts |
| RFC-Q-08 | docs/RFC.md | Questão em aberto | Criptografia da secret em repouso e versionamento de payload → revisão de segurança | TRANSCRICAO | [09:46] Sofia |
| RFC-RSK-01 | docs/RFC.md | Risco | Worker parado acumula eventos | TRANSCRICAO | [09:11] Diego |
| RFC-RSK-02 | docs/RFC.md | Risco | Cliente lento segura a fila (single-worker) | TRANSCRICAO | [09:42] Diego |
| RFC-RSK-03 | docs/RFC.md | Risco | Vazamento de secret | TRANSCRICAO | [09:22] Diego |
| RFC-RSK-04 | docs/RFC.md | Risco | Cliente processa duplicata | TRANSCRICAO | [09:25] Sofia |
| RFC-RSK-05 | docs/RFC.md | Risco | Prazo de 3 sprints contra fim de novembro | TRANSCRICAO | [09:45] Marcos |
| RFC-Q-09 | docs/RFC.md | Questão em aberto | SSRF: HTTPS não impede URL para rede interna (severidade crítica) | TRANSCRICAO | [09:19] Sofia |
| RFC-Q-10 | docs/RFC.md | Questão em aberto | Retry por status HTTP (4xx, 429 com Retry-After) | TRANSCRICAO | [09:42] Diego |
| RFC-Q-11 | docs/RFC.md | Questão em aberto | Versionamento do payload congelado na outbox | TRANSCRICAO | [09:52] Larissa |
| RFC-RSK-06 | docs/RFC.md | Proposta (revisão humana) | Risco de SSRF pelo worker | TRANSCRICAO | [09:19] Sofia |
| RFC-RSK-07 | docs/RFC.md | Risco | Acesso a webhooks de outro cliente; isolamento recomendado como pré-condição | TRANSCRICAO | [09:37] Sofia |

## ADRs — [docs/adrs/](adrs/README.md)

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox no MySQL, inserção na mesma transação | TRANSCRICAO | [09:06] Diego |
| ADR-001-CTX | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | Transação de changeStatus e ausência de broker | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-DEC-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Falha ao inserir na outbox → rollback | TRANSCRICAO | [09:40] Bruno |
| ADR-001-DEC-03 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Filtro de status na inserção | TRANSCRICAO | [09:34] Bruno |
| ADR-001-ALT-02 | docs/adrs/ADR-001-outbox-no-mysql.md | Alternativa descartada | Redis Streams | TRANSCRICAO | [09:07] Larissa |
| ADR-001-CON-01 | docs/adrs/ADR-001-outbox-no-mysql.md | Trade-off | Crescimento da tabela; arquivamento fora de escopo | TRANSCRICAO | [09:08] Diego |
| ADR-002 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Worker em processo separado | TRANSCRICAO | [09:11] Diego |
| ADR-002-DEC-02 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | src/worker.ts + npm run worker | TRANSCRICAO | [09:11] Larissa |
| ADR-002-DEC-03 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | PrismaClient próprio, mesma DATABASE_URL | TRANSCRICAO | [09:30] Bruno |
| ADR-002-DEC-04 | docs/adrs/ADR-002-worker-separado-em-polling.md | Decisão | Single-worker; ordem por order_id como limitação conhecida | TRANSCRICAO | [09:13] Larissa |
| ADR-002-CTX | docs/adrs/ADR-002-worker-separado-em-polling.md | Contexto | createPrismaClient() reutilizado pelo worker | CODIGO | src/config/database.ts |
| ADR-002-ALT-02 | docs/adrs/ADR-002-worker-separado-em-polling.md | Alternativa descartada | Múltiplos workers (adiado) | TRANSCRICAO | [09:13] Diego |
| ADR-003 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Decisão | 5 tentativas com backoff 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| ADR-003-INT-01 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Inferência | "Quase 15h" ⇒ 1 envio inicial + 5 retentativas (6 requisições) | TRANSCRICAO | [09:17] Diego |
| ADR-003-INT-02 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Inferência | "Cinco cobre 12 ou 24 horas" só fecha com 5 intervalos de retry | TRANSCRICAO | [09:15] Diego |
| ADR-003-INT-03 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Inferência | "Retentaria três vezes em 30 minutos": tentativa = retentativa | TRANSCRICAO | [09:16] Diego |
| ADR-003-INT-04 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Decisão | Resumo final: 5 intervalos de retry com "total 5 tentativas" | TRANSCRICAO | [09:48] Larissa |
| ADR-003-DEC-02 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Decisão | DLQ em tabela separada webhook_dead_letter | TRANSCRICAO | [09:18] Diego |
| ADR-003-DEC-03 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Proposta | Payload > 64KB vai direto para a DLQ (validado no worker) | TRANSCRICAO | [09:23] Sofia |
| ADR-003-ALT-01 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Alternativa descartada | Retry indefinido | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Alternativa descartada | 3 tentativas | TRANSCRICAO | [09:16] Bruno |
| ADR-003-ALT-03 | docs/adrs/ADR-003-retry-backoff-e-dlq.md | Alternativa descartada | DLQ como status failed na outbox | TRANSCRICAO | [09:17] Larissa |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, header X-Signature | TRANSCRICAO | [09:20] Sofia |
| ADR-004-DEC-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Secret única por endpoint | TRANSCRICAO | [09:21] Sofia |
| ADR-004-DEC-03 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Rotação com grace period de 24h | TRANSCRICAO | [09:22] Sofia |
| ADR-004-CTX | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Contexto | Senhas usam bcrypt (irreversível); secret de HMAC precisa ser recuperável | CODIGO | src/modules/auth/auth.service.ts |
| ADR-004-ALT-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Alternativa (plausível) | Depender só de TLS: não prova origem ao cliente | TRANSCRICAO | [09:19] Sofia |
| ADR-004-PEN-01 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Proposta (revisão humana) | Secret cifrada em repouso (AES-256-GCM), pendente de aprovação da Sofia | TRANSCRICAO | [09:46] Sofia |
| ADR-004-PEN-02 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Proposta (revisão humana) | Validação de destino (SSRF) como complemento à assinatura | TRANSCRICAO | [09:19] Sofia |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | At-least-once; cliente pode receber duplicado | TRANSCRICAO | [09:24] Diego |
| ADR-005-DEC-02 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | X-Event-Id = UUID gerado na entrada da outbox | TRANSCRICAO | [09:25] Diego |
| ADR-005-DEC-03 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | X-Webhook-Id identifica o cadastro | TRANSCRICAO | [09:45] Diego |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso máximo: AppError, Pino, error middleware, módulos, Zod | TRANSCRICAO | [09:30] Larissa |
| ADR-006-DEC-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Módulo src/modules/webhooks com controller/service/repository/routes/schemas | TRANSCRICAO | [09:27] Bruno |
| ADR-006-DEC-03 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Prefixo WEBHOOK_ nos códigos de erro | TRANSCRICAO | [09:29] Larissa |
| ADR-006-DEC-04 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Restrição | Sem logger novo; middleware de erro sem mudanças | TRANSCRICAO | [09:29] Bruno |
| ADR-006-NUA-01 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Inferência | NotFoundError fixa o código NOT_FOUND → WebhookNotFoundError estende AppError | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-NUA-02 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Inferência | validate.middleware converte ZodError em VALIDATION_ERROR | CODIGO | src/middlewares/validate.middleware.ts |
| ADR-006-NUA-03 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Inferência | Logger com base.service fixo → child logger no worker | CODIGO | src/shared/logger/index.ts |
| ADR-007 | docs/adrs/ADR-007-payload-snapshot-na-insercao.md | Decisão | Payload renderizado (snapshot) na inserção | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-01 | docs/adrs/ADR-007-payload-snapshot-na-insercao.md | Alternativa descartada | Guardar só order_id e renderizar no envio | TRANSCRICAO | [09:51] Bruno |
| ADR-007-ALT-02 | docs/adrs/ADR-007-payload-snapshot-na-insercao.md | Alternativa descartada | Incluir itens no payload | TRANSCRICAO | [09:43] Diego |

## FDD — [docs/FDD.md](FDD.md)

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| FDD-OT-01 | docs/FDD.md | Objetivo técnico | Commit ⇒ evento; rollback ⇒ sem evento | TRANSCRICAO | [09:06] Diego |
| FDD-OT-02 | docs/FDD.md | Objetivo técnico | Entrega < 10s no caminho feliz | TRANSCRICAO | [09:10] Larissa |
| FDD-OT-03 | docs/FDD.md | Objetivo técnico | Nenhum HTTP externo dentro da transação | TRANSCRICAO | [09:04] Bruno |
| FDD-OT-04 | docs/FDD.md | Objetivo técnico | Tolerância de ~14h36 antes da DLQ | TRANSCRICAO | [09:17] Diego |
| FDD-OT-06 | docs/FDD.md | Restrição | Zero dependências npm novas | TRANSCRICAO | [09:29] Bruno |
| FDD-ESC-01 | docs/FDD.md | Fora de escopo | Evento na criação do pedido não é emitido nesta fase | CODIGO | src/modules/orders/order.service.ts |
| FDD-COMP-01 | docs/FDD.md | Decisão | Processor em src/modules/webhooks/webhook.processor.ts | TRANSCRICAO | [09:28] Bruno |
| FDD-COMP-02 | docs/FDD.md | Proposta | Nomes webhook.publisher.ts e webhook.signature.ts | TRANSCRICAO | [09:41] Bruno |
| FDD-DADOS-01 | docs/FDD.md | Decisão | webhook_endpoints: url, secret, customer_id, ativo | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-02 | docs/FDD.md | Decisão | Status da outbox: pendente, processando, falhou, entregue | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-03 | docs/FDD.md | Decisão | IDs UUID (padrão do projeto) | TRANSCRICAO | [09:51] Larissa |
| FDD-DADOS-04 | docs/FDD.md | Restrição | Padrão de ids Char(36) uuid() no schema | CODIGO | prisma/schema.prisma |
| FDD-DADOS-05 | docs/FDD.md | Decisão | DLQ com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-06 | docs/FDD.md | Inferência | Tabela webhook_deliveries necessária para o histórico | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-07 | docs/FDD.md | Proposta | Uma linha de outbox por (evento, endpoint) | TRANSCRICAO | [09:25] Diego |
| FDD-DADOS-08 | docs/FDD.md | Proposta | DELETE de webhook em cascata (outbox, entregas, DLQ) | TRANSCRICAO | [09:33] Bruno |
| FDD-FLUXO-01 | docs/FDD.md | Decisão | publishWebhookEvent chamado em changeStatus dentro do tx | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-02 | docs/FDD.md | Decisão | Função pura recebendo tx, sem injetar repository | TRANSCRICAO | [09:41] Diego |
| FDD-FLUXO-03 | docs/FDD.md | Decisão | Filtro na inserção: sem interessado, não insere | TRANSCRICAO | [09:34] Bruno |
| FDD-FLUXO-04 | docs/FDD.md | Decisão | Snapshot do payload na inserção | TRANSCRICAO | [09:52] Diego |
| FDD-FLUXO-05 | docs/FDD.md | Decisão | Polling a cada 2s | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-06 | docs/FDD.md | Decisão | Lote pequeno de pendentes mais antigos | TRANSCRICAO | [09:08] Diego |
| FDD-FLUXO-07 | docs/FDD.md | Decisão | Processamento em ordem de created_at (single-worker) | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-08 | docs/FDD.md | Proposta | Claim condicional UPDATE ... WHERE status=PENDING | TRANSCRICAO | [09:13] Diego |
| FDD-FLUXO-09 | docs/FDD.md | Proposta | Lease de 60s para recuperar PROCESSING após queda do worker | TRANSCRICAO | [09:11] Diego |
| FDD-FLUXO-10 | docs/FDD.md | Decisão | Timeout HTTP de 10s → retry | TRANSCRICAO | [09:42] Diego |
| FDD-FLUXO-11 | docs/FDD.md | Decisão | Tabela de intervalos 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-12 | docs/FDD.md | Decisão | Falha permanente → DLQ após o teto de tentativas | TRANSCRICAO | [09:15] Diego |
| FDD-FLUXO-13 | docs/FDD.md | Decisão | Replay recoloca na outbox como pendente | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-14 | docs/FDD.md | Decisão | Replay exige ADMIN via requireRole existente | TRANSCRICAO | [09:36] Larissa |
| FDD-FLUXO-15 | docs/FDD.md | Requisito Não Funcional | Log de quem fez o replay | TRANSCRICAO | [09:36] Sofia |
| FDD-FLUXO-16 | docs/FDD.md | Decisão | Rotação: antiga válida 24h em paralelo | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-17 | docs/FDD.md | Proposta | Dupla assinatura em X-Signature durante o grace | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-01 | docs/FDD.md | Decisão | Headers X-Event-Id, X-Signature, X-Timestamp, Content-Type | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-02 | docs/FDD.md | Decisão | Header X-Webhook-Id | TRANSCRICAO | [09:44] Sofia |
| FDD-CONTRATO-03 | docs/FDD.md | Decisão | Payload: event_id, event_type, timestamp, order_id, order_number, from/to_status, customer_id, total_cents | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-04 | docs/FDD.md | Proposta | Formato `sha256=<hex>` da assinatura | TRANSCRICAO | [09:20] Sofia |
| FDD-CONTRATO-05 | docs/FDD.md | Requisito Funcional | POST /api/v1/webhooks (url, statuses; secret na resposta) | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-06 | docs/FDD.md | Proposta | customerId no body (POST) e em query (GET), resolvendo Q-2 | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRATO-07 | docs/FDD.md | Requisito Funcional | GET /api/v1/webhooks (lista por customer) | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-08 | docs/FDD.md | Requisito Funcional | PATCH /api/v1/webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-09 | docs/FDD.md | Requisito Funcional | DELETE /api/v1/webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-10 | docs/FDD.md | Requisito Funcional | POST /api/v1/webhooks/:id/rotate-secret (caminho: proposta) | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-11 | docs/FDD.md | Requisito Funcional | GET /api/v1/webhooks/:id/deliveries (até 100) | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-12 | docs/FDD.md | Requisito Funcional | POST /api/v1/admin/webhooks/dead-letter/:id/replay | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-13 | docs/FDD.md | Restrição | CRUD liberado para qualquer role autenticada | TRANSCRICAO | [09:37] Sofia |
| FDD-CONTRATO-14 | docs/FDD.md | Requisito Não Funcional | URL https validada no schema Zod | TRANSCRICAO | [09:23] Sofia |
| FDD-CONTRATO-15 | docs/FDD.md | Restrição | Rotas montadas sob /api/v1 | CODIGO | src/app.ts |
| FDD-CONTRATO-16 | docs/FDD.md | Restrição | Envelope de erro { error: { code, message, details } } | CODIGO | src/middlewares/error.middleware.ts |
| FDD-CONTRATO-17 | docs/FDD.md | Restrição | Valores de to_status alcançáveis: PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED | CODIGO | src/modules/orders/order.status.ts |
| FDD-CONTRATO-18 | docs/FDD.md | Restrição | Paginação { data, pagination } | CODIGO | src/shared/http/response.ts |
| FDD-CONTRATO-19 | docs/FDD.md | Requisito Funcional | Cliente consulta GET /orders/:id para detalhes | TRANSCRICAO | [09:43] Diego |
| FDD-ERR-01 | docs/FDD.md | Decisão | Prefixo WEBHOOK_ em todos os códigos do módulo | TRANSCRICAO | [09:29] Larissa |
| FDD-ERR-02 | docs/FDD.md | Decisão | WEBHOOK_NOT_FOUND e WEBHOOK_INVALID_URL como exemplos | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-03 | docs/FDD.md | Inferência | WEBHOOK_INVALID_URL sai como VALIDATION_ERROR com código em details | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-ERR-04 | docs/FDD.md | Restrição | Classes de erro em http-errors.ts, exportadas por index.ts | CODIGO | src/shared/errors/http-errors.ts |
| FDD-ERR-05 | docs/FDD.md | Requisito Não Funcional | WEBHOOK_PAYLOAD_TOO_LARGE: erra em vez de truncar | TRANSCRICAO | [09:23] Sofia |
| FDD-ERR-06 | docs/FDD.md | Requisito Não Funcional | WEBHOOK_DELIVERY_TIMEOUT após 10s | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-07 | docs/FDD.md | Decisão | WEBHOOK_MAX_ATTEMPTS_EXCEEDED → DLQ | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-08 | docs/FDD.md | Decisão | WEBHOOK_OUTBOX_INSERT_FAILED → rollback da mudança de status | TRANSCRICAO | [09:41] Diego |
| FDD-ERR-09 | docs/FDD.md | Proposta | WEBHOOK_DLQ_ENTRY_NOT_FOUND / WEBHOOK_DLQ_ALREADY_REPLAYED | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-10 | docs/FDD.md | Proposta | WEBHOOK_ENDPOINT_INACTIVE (descarte/replay bloqueado) | TRANSCRICAO | [09:21] Bruno |
| FDD-ERR-11 | docs/FDD.md | Restrição | Falha não tratada responde 500 INTERNAL_SERVER_ERROR | CODIGO | src/middlewares/error.middleware.ts |
| FDD-RES-01 | docs/FDD.md | Requisito Não Funcional | Limite de 64KB | TRANSCRICAO | [09:24] Diego |
| FDD-RES-02 | docs/FDD.md | Restrição | Graceful shutdown SIGINT/SIGTERM + $disconnect | CODIGO | src/server.ts |
| FDD-RES-03 | docs/FDD.md | Trade-off | Backoff sem jitter (single-worker) | TRANSCRICAO | [09:12] Diego |
| FDD-SEG-01 | docs/FDD.md | Decisão | Secret gerada pela plataforma | TRANSCRICAO | [09:31] Marcos |
| FDD-SEG-02 | docs/FDD.md | Proposta | redactPaths acrescido de *.secret e *.previousSecret | CODIGO | src/shared/logger/index.ts |
| FDD-SEG-03 | docs/FDD.md | Restrição | Revisão de segurança ≥ 2 dias úteis | TRANSCRICAO | [09:46] Sofia |
| FDD-SEG-04 | docs/FDD.md | Requisito Não Funcional | Cliente valida origem e integridade | TRANSCRICAO | [09:19] Sofia |
| FDD-SEG-05 | docs/FDD.md | Inferência | Papéis existentes apenas ADMIN e OPERATOR | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-OBS-01 | docs/FDD.md | Restrição | Logs com o Pino existente | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | docs/FDD.md | Métrica | Latência ponta a ponta p95 < 10s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-03 | docs/FDD.md | Proposta | Métricas derivadas de logs e SQL, sem infraestrutura nova | TRANSCRICAO | [09:07] Diego |
| FDD-OBS-04 | docs/FDD.md | Proposta | Correlação requestId → orderId → eventId | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-OBS-05 | docs/FDD.md | Proposta | Volume por endpoint como insumo para rate limiting | TRANSCRICAO | [09:39] Larissa |
| FDD-OBS-06 | docs/FDD.md | Requisito Não Funcional | Log de auditoria do replay (webhook_dlq_replayed) | TRANSCRICAO | [09:36] Sofia |
| FDD-DEP-01 | docs/FDD.md | Dependência | Node ≥ 20 (fetch, AbortSignal.timeout); uuid já presente | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Dependência | MySQL 8 existente | CODIGO | docker-compose.yml |
| FDD-DEP-03 | docs/FDD.md | Proposta | Variáveis WEBHOOK_* no envSchema | CODIGO | src/config/env.ts |
| FDD-DEP-04 | docs/FDD.md | Decisão | Mesmo banco e mesma stack para o worker | TRANSCRICAO | [09:11] Diego |
| FDD-COMPAT-01 | docs/FDD.md | Restrição | Nunca mais de uma instância do worker | TRANSCRICAO | [09:12] Diego |
| FDD-INT-01 | docs/FDD.md | Integração | changeStatus chama publishWebhookEvent após orderStatusHistory.create | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Máquina de estados define os status filtráveis | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | docs/FDD.md | Integração | Novos modelos e relações inversas | CODIGO | prisma/schema.prisma |
| FDD-INT-04 | docs/FDD.md | Integração | Classes Webhook*Error exportadas | CODIGO | src/shared/errors/index.ts |
| FDD-INT-05 | docs/FDD.md | Integração | error.middleware sem alteração | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-06 | docs/FDD.md | Integração | validate() reutilizado | TRANSCRICAO | [09:30] Larissa |
| FDD-INT-07 | docs/FDD.md | Integração | authenticate / requireRole('ADMIN') | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-08 | docs/FDD.md | Integração | buildControllers instancia o módulo webhooks | CODIGO | src/app.ts |
| FDD-INT-09 | docs/FDD.md | Integração | buildApiRouter monta /webhooks e /admin/webhooks | CODIGO | src/routes/index.ts |
| FDD-INT-10 | docs/FDD.md | Integração | src/worker.ts modelado sobre src/server.ts | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-11 | docs/FDD.md | Integração | Worker usa createPrismaClient() | CODIGO | src/config/database.ts |
| FDD-INT-12 | docs/FDD.md | Integração | Script npm run worker | CODIGO | package.json |
| FDD-INT-13 | docs/FDD.md | Integração | Limpeza das novas tabelas no setup de testes | CODIGO | tests/setup.ts |
| FDD-INT-14 | docs/FDD.md | Integração | Fábrica createTestWebhook | CODIGO | tests/helpers/factories.ts |
| FDD-INT-15 | docs/FDD.md | Integração | Testes no padrão Supertest + MySQL real | CODIGO | tests/orders.test.ts |
| FDD-ERR-12 | docs/FDD.md | Proposta | WEBHOOK_CUSTOMER_NOT_FOUND quando o customerId do body não existe | TRANSCRICAO | [09:32] Larissa |
| FDD-ERR-13 | docs/FDD.md | Proposta | WEBHOOK_DELIVERY_HTTP_ERROR / WEBHOOK_DELIVERY_NETWORK_ERROR como falhas retentáveis | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-14 | docs/FDD.md | Proposta | WEBHOOK_DELIVERY_INTERRUPTED quando o worker cai durante o envio | TRANSCRICAO | [09:11] Diego |
| FDD-AC-01 | docs/FDD.md | Critério de aceite técnico | 1 linha PENDING por webhook interessado | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-02 | docs/FDD.md | Critério de aceite técnico | Sem webhook interessado, nenhuma linha criada | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-03 | docs/FDD.md | Critério de aceite técnico | Erro na outbox ⇒ rollback de status, histórico e estoque | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-04 | docs/FDD.md | Critério de aceite técnico | Entrega 2xx ⇒ DELIVERED com os 5 headers e assinatura válida | TRANSCRICAO | [09:44] Diego |
| FDD-AC-05 | docs/FDD.md | Critério de aceite técnico | Resposta 500 ⇒ PENDING com attempts=1 e próximo envio em +60s | TRANSCRICAO | [09:17] Diego |
| FDD-AC-06 | docs/FDD.md | Critério de aceite técnico | Resposta após 10s ⇒ WEBHOOK_DELIVERY_TIMEOUT | TRANSCRICAO | [09:42] Diego |
| FDD-AC-07 | docs/FDD.md | Critério de aceite técnico | Após a 6ª falha: FAILED + registro na DLQ | TRANSCRICAO | [09:17] Diego |
| FDD-AC-08 | docs/FDD.md | Critério de aceite técnico | Payload > 65.536 bytes ⇒ DLQ sem requisição HTTP | TRANSCRICAO | [09:24] Larissa |
| FDD-AC-09 | docs/FDD.md | Critério de aceite técnico | Replay: OPERATOR 403, ADMIN 202 + log | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-10 | docs/FDD.md | Critério de aceite técnico | URL http:// ⇒ 400 | TRANSCRICAO | [09:23] Sofia |
| FDD-AC-11 | docs/FDD.md | Critério de aceite técnico | Secret só nas respostas de criação e rotação; nunca em listagem ou log | TRANSCRICAO | [09:31] Marcos |
| FDD-AC-12 | docs/FDD.md | Critério de aceite técnico | Dupla assinatura por 24h após a rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-AC-13 | docs/FDD.md | Critério de aceite técnico | X-Event-Id idêntico em retry e replay | TRANSCRICAO | [09:25] Diego |
| FDD-AC-14 | docs/FDD.md | Critério de aceite técnico | Ordem por pedido preservada com receptor saudável | TRANSCRICAO | [09:12] Diego |
| FDD-AC-15 | docs/FDD.md | Critério de aceite técnico | Deliveries: ≤ 100 itens, ordem decrescente, com payload e resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-AC-16 | docs/FDD.md | Critério de aceite técnico | p95 de deliveredAt − createdAt < 10s em homologação | TRANSCRICAO | [09:02] Marcos |
| FDD-RSK-01 | docs/FDD.md | Risco | Head-of-line blocking por cliente lento | TRANSCRICAO | [09:42] Diego |
| FDD-RSK-02 | docs/FDD.md | Risco | Crescimento da outbox sem arquivamento | TRANSCRICAO | [09:07] Bruno |
| FDD-RSK-03 | docs/FDD.md | Risco | Burst de eventos sem rate limit | TRANSCRICAO | [09:38] Diego |
| FDD-SEG-06 | docs/FDD.md | Proposta (revisão humana) | SSRF: bloquear loopback, privados, link-local e multicast no cadastro e no envio | TRANSCRICAO | [09:19] Sofia |
| FDD-SEG-07 | docs/FDD.md | Proposta (revisão humana) | IP pinning via lookup do node:https contra DNS rebinding; fetch descartado | CODIGO | package.json |
| FDD-SEG-08 | docs/FDD.md | Proposta (revisão humana) | Regra de egress na infraestrutura (defesa em profundidade) | TRANSCRICAO | [09:19] Sofia |
| FDD-SEG-09 | docs/FDD.md | Proposta (revisão humana) | Secret cifrada com AES-256-GCM; chave em WEBHOOK_SECRET_ENCRYPTION_KEY | TRANSCRICAO | [09:46] Sofia |
| FDD-SEG-10 | docs/FDD.md | Proposta (revisão humana) | Logs de auditoria de criação, edição, remoção e rotação | TRANSCRICAO | [09:36] Sofia |
| FDD-SEG-11 | docs/FDD.md | Restrição | Isolamento por customer é recomendação, não requisito: CRUD aberto "por enquanto" | TRANSCRICAO | [09:37] Sofia |
| FDD-SEG-12 | docs/FDD.md | Inferência | Sem vínculo User ↔ Customer, o isolamento não é implementável hoje | CODIGO | prisma/schema.prisma |
| FDD-SEG-13 | docs/FDD.md | Inferência | Validação de destino no service, porque validate() usa parse síncrono e DNS é assíncrono | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-ERR-15 | docs/FDD.md | Proposta (revisão humana) | WEBHOOK_DESTINATION_BLOCKED: 422 no cadastro; DLQ sem retry no envio | TRANSCRICAO | [09:19] Sofia |
| FDD-RES-04 | docs/FDD.md | Proposta (revisão humana) | Corpo da resposta lido em stream até 4 KB | TRANSCRICAO | [09:34] Marcos |
| FDD-RES-05 | docs/FDD.md | Proposta (revisão humana) | Timeout de 10s cobre conexão, TLS e resposta; keepAlive para reuso de conexões | TRANSCRICAO | [09:42] Diego |
| FDD-COMPAT-02 | docs/FDD.md | Restrição | Variável obrigatória aborta o boot se ausente (loadEnv) | CODIGO | src/config/env.ts |
| FDD-AC-17 | docs/FDD.md | Critério de aceite técnico | Queda depois do 2xx: reenvio com o mesmo X-Event-Id após o lease | TRANSCRICAO | [09:24] Diego |
| FDD-AC-18 | docs/FDD.md | Critério de aceite técnico | SSRF bloqueado no cadastro (422) e no envio (DLQ sem conexão) | TRANSCRICAO | [09:19] Sofia |
| FDD-AC-19 | docs/FDD.md | Critério de aceite técnico | Secret cifrada; resposta limitada a 4 KB; logs de auditoria | TRANSCRICAO | [09:46] Sofia |
| FDD-RSK-04 | docs/FDD.md | Proposta (revisão humana) | Risco de SSRF (severidade crítica) | TRANSCRICAO | [09:19] Sofia |
| FDD-RSK-05 | docs/FDD.md | Risco | Usuário de um cliente acessa webhooks de outro | TRANSCRICAO | [09:37] Sofia |

## Resumo de cobertura

Números obtidos com o script de validação descrito no [README](../README.md#iterações-e-ajustes) (rodada final).

- **Linhas no tracker:** 290.
- **Fonte = TRANSCRICAO:** 248 linhas (85,5%; o exigido é ≥ 70%).
- **Fonte = CODIGO:** 42 linhas (o exigido é ≥ 5).
- **Localizações `[hh:mm] Nome` que existem na transcrição com aquele falante:** 100%.
- **Caminhos CODIGO que existem no repositório:** 100%.
- **Itens com ID nos documentos que têm linha no tracker:** 100% (o exigido é ≥ 80%). São eles: PRD-FR (14), PRD-NFR (11), PRD-OBJ (5), PRD-OOS (7), PRD-RSK (8), PRD-FUT (2), PRD-AC (10), RFC-ALT (6), RFC-Q (11), ADRs (7), FDD-AC (19), endpoints do FDD (8) e códigos `WEBHOOK_*` da matriz de erros (14).
- **Linhas do tipo Proposta** (sem decisão na reunião): 36, todas marcadas como tal. Destas, 21 foram criadas na produção do pacote e 15 vieram da revisão humana, marcadas como "Proposta (revisão humana)".
