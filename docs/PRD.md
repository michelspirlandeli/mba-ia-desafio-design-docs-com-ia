# PRD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
|---|---|
| **Product Manager** | Marcos |
| **Tech Lead** | Larissa |
| **Engenharia** | Bruno (Pedidos), Diego (Plataforma) |
| **Segurança** | Sofia |
| **Status** | Aprovado em reunião; documentação em revisão |
| **Data** | 2026-11-16 |
| **Prazo** | 3 sprints, incluindo revisão de segurança ([09:47] Larissa); expectativa da Atlas: fim de novembro ([09:45] Marcos) |
| **Documentos relacionados** | [RFC](RFC.md) · [FDD](FDD.md) · [ADRs](adrs/README.md) · [TRACKER](TRACKER.md) |

> Os itens marcados **(proposta)** não foram definidos na reunião. São sugestões explícitas deste documento e precisam de validação do PM.

## 1. Resumo e contexto da feature

O OMS passa a **avisar automaticamente os sistemas dos clientes B2B quando o status de um pedido muda**. O aviso é uma chamada HTTPS (webhook) para a URL cadastrada pelo cliente. Cada cliente escolhe quais mudanças de status quer receber, verifica pela assinatura que a mensagem veio do OMS e consulta o histórico de entregas.

Hoje o OMS gerencia o ciclo de vida do pedido (`PENDING → PAID → PROCESSING → SHIPPED → DELIVERED`, ou `CANCELLED`), mas **não tem nenhum mecanismo de notificação externa**. Quem integra precisa perguntar periodicamente.

## 2. Problema e motivação

**Problema.** Três clientes B2B (**Atlas Comercial**, **MaxDistribuição** e **Nova Cargo**) fazem polling em `GET /orders` para descobrir mudanças de status. Para eles, a integração fica **lenta e cara** ([09:00] Marcos).

**Motivação de negócio**

- Pedido formal dos três clientes na semana anterior à reunião ([09:00] Marcos).
- **Risco de churn:** a Atlas sinalizou que pode migrar para um concorrente se a solução não sair até o fim do trimestre ([09:00] Marcos).
- Diminuir a carga de polling que esses clientes geram na API.

## 3. Público-alvo e cenários de uso

| Persona | Quem é | O que precisa |
|---|---|---|
| **Integrador B2B** | Sistema do cliente (Atlas, MaxDistribuição, Nova Cargo) | Receber, em menos de 10s, a mudança de status dos seus pedidos, com garantia de origem ([09:02] Marcos, [09:19] Sofia) |
| **Usuário que representa o cliente** | Usuário autenticado no OMS que configura a integração pela API ([09:32] Marcos) | Cadastrar, editar, remover e listar webhooks; rotacionar a secret; ver o histórico de entregas |
| **Administrador do OMS** (role `ADMIN`) | Operação da plataforma | Reprocessar entregas que falharam definitivamente, com rastro de quem fez ([09:36] Sofia) |
| **PM / Portal do desenvolvedor** | Marcos | Documentar o contrato (deduplicação, assinatura) para os clientes ([09:26] Marcos, [09:40] Marcos) |

**Cenários**

1. **Notificação de envio.** A Atlas cadastra um webhook que assina `SHIPPED` e `DELIVERED`. Quando o operador marca o pedido como `SHIPPED`, o sistema da Atlas recebe o aviso em segundos ([09:33] Marcos).
2. **Cliente em manutenção.** O endpoint da MaxDistribuição fica 2h fora do ar ([09:16] Diego). O OMS tenta de novo ao longo de ~15h e entrega quando o endpoint volta, sem ação de ninguém.
3. **Falha definitiva.** Um endpoint fica fora por mais de 15h. O evento vai para uma fila de falhas e um ADMIN o reenvia depois que o cliente corrige o problema ([09:18] Diego).
4. **Secret vazada.** O cliente vaza a secret em log ([09:22] Diego), pede uma nova pela API e tem 24h para atualizar seus sistemas sem perder eventos ([09:21] Sofia).
5. **Diagnóstico.** O cliente reclama que "não recebeu" e consulta as últimas entregas, com status HTTP, tempo de resposta e payload ([09:34] Marcos).

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta | Origem |
|---|---|---|---|---|
| PRD-OBJ-01 | Notificar em "tempo real" na percepção do cliente | Tempo entre a mudança de status e a entrega confirmada (p95, primeira tentativa) | **< 10 segundos** | [09:02] Marcos; [09:10] Larissa |
| PRD-OBJ-02 | Atender os clientes que pediram a feature | Clientes solicitantes com webhook ativo em produção | **3 de 3** (Atlas, MaxDistribuição, Nova Cargo) | [09:00] Marcos |
| PRD-OBJ-03 | Cumprir o prazo comprometido | Entrega em produção | **≤ 3 sprints**, com revisão de segurança incluída | [09:47] Larissa |
| PRD-OBJ-04 | Não perder notificações | % de eventos entregues sem precisar de replay manual | **≥ 99%** **(proposta)** | Sem meta na reunião |
| PRD-OBJ-05 | Reduzir o polling dos clientes | Chamadas a `GET /orders` dos usuários desses clientes, 30 dias antes e depois | **Redução mensurável** (linha de base a coletar) **(proposta)** | Problema citado em [09:00] Marcos |

## 5. Escopo

### 5.1 Incluso

- Notificação **de saída** a cada mudança de status de pedido ([09:02] Marcos).
- Cadastro, edição, remoção e listagem de webhooks por customer, com filtro de status ([09:31]–[09:33]).
- Secret única por webhook, gerada pela plataforma, e rotação com 24h de convivência ([09:21] Sofia).
- Assinatura de cada mensagem para o cliente verificar a origem ([09:20] Sofia).
- Novas tentativas automáticas por ~15h e fila de falhas definitivas com reenvio manual por ADMIN ([09:17]–[09:18]).
- Histórico de entregas por webhook ([09:34] Marcos).

### 5.2 Fora de escopo

| ID | Item | Situação | Origem |
|---|---|---|---|
| PRD-OOS-01 | **Aviso por e-mail ao cliente quando o webhook falha** (ex.: 3 falhas seguidas) | **Adiado** para a próxima fase, depois de medir o impacto | [09:37] Larissa, [09:38] Marcos |
| PRD-OOS-02 | **Dashboard visual** para o cliente ver seus webhooks | **Descartado** nesta feature; é projeto do time de frontend. Nesta fase, apenas API | [09:40] Larissa |
| PRD-OOS-03 | **Rate limiting** de envio por cliente | **Adiado**: "observar e decidir depois" | [09:39] Larissa |
| PRD-OOS-04 | Webhooks **de entrada** (cliente enviando para o OMS) | **Descartado**: só saída | [09:02] Marcos |
| PRD-OOS-05 | Garantia de **ordem global** dos eventos | **Descartado**: os clientes não pediram | [09:14] Marcos |
| PRD-OOS-06 | Arquivamento de eventos antigos (~30 dias) | **Fora** desta feature | [09:08] Diego |
| PRD-OOS-07 | Restringir quais usuários podem configurar webhooks | **Adiado** ("mais pra frente a gente pode endurecer") | [09:37] Sofia |

### 5.3 Evoluções sugeridas na revisão (não discutidas na reunião)

Ideias registradas para avaliação futura. **Não são requisitos desta fase.**

| ID | Evolução | Motivação |
|---|---|---|
| PRD-FUT-01 | **Saúde do endpoint:** indicador de endpoints com falhas recorrentes, consultável pelo cliente | Complementa o e-mail de alerta adiado ([09:37] Marcos), sem exigir canal novo |
| PRD-FUT-02 | **Evento de teste:** o cliente dispara um evento fictício para validar a integração (assinatura e deduplicação) | Facilita o onboarding documentado no portal ([09:40] Marcos) |

E-mail de alerta, rate limiting e dashboard, que também são evoluções naturais, já estão listados em §5.2 como adiados ou descartados na reunião.

## 6. Requisitos funcionais

| ID | Requisito | Prioridade | Origem |
|---|---|---|---|
| PRD-FR-001 | O sistema deve notificar o webhook do cliente sempre que um pedido dele mudar de status | Must | [09:00] Marcos |
| PRD-FR-002 | O usuário deve poder **cadastrar** um webhook informando o customer, a URL e a lista de status de interesse. A **secret é gerada pela plataforma e devolvida na criação** | Must | [09:31] Marcos |
| PRD-FR-003 | O usuário deve poder **editar** um webhook (URL, status de interesse, ativo/inativo) | Must | [09:33] Bruno |
| PRD-FR-004 | O usuário deve poder **remover** um webhook | Must | [09:33] Bruno |
| PRD-FR-005 | O usuário deve poder **listar** os webhooks de um customer | Must | [09:33] Bruno |
| PRD-FR-006 | Cada webhook só recebe as mudanças para os status que escolheu (ex.: só `SHIPPED` e `DELIVERED`) | Must | [09:33] Marcos |
| PRD-FR-007 | O usuário deve poder **solicitar uma nova secret**; a anterior continua válida por **24h** | Must | [09:21] Sofia |
| PRD-FR-008 | O usuário deve poder consultar o **histórico das últimas 100 entregas** de um webhook, com sucesso/falha, payload, resposta e tempo de resposta | Must | [09:34] Marcos |
| PRD-FR-009 | Se a entrega falhar, o sistema deve **tentar de novo automaticamente** até 5 vezes, com intervalos de 1m, 5m, 30m, 2h e 12h | Must | [09:17] Larissa |
| PRD-FR-010 | Esgotadas as tentativas, o evento deve ir para uma **fila de falhas (DLQ)** com o motivo registrado | Must | [09:18] Diego |
| PRD-FR-011 | Um **ADMIN** deve poder **reenviar** um evento da DLQ, ficando registrado quem fez | Must | [09:18] Diego, [09:36] Sofia |
| PRD-FR-012 | Cada mensagem deve levar um **identificador único do evento** e o **identificador do webhook**, para o cliente deduplicar e saber de qual cadastro veio | Must | [09:25] Diego, [09:44] Sofia |
| PRD-FR-013 | A mensagem deve conter os dados essenciais da mudança (pedido, número, status anterior e novo, customer, total, data/hora), **sem os itens** | Must | [09:43] Diego |
| PRD-FR-014 | Um webhook pode ser **ativado/desativado** sem ser removido | Should | [09:21] Bruno |

## 7. Requisitos não funcionais

| ID | Requisito | Origem |
|---|---|---|
| PRD-NFR-001 | **Latência:** notificação em menos de 10s no caminho feliz | [09:02] Marcos |
| PRD-NFR-002 | **Consistência:** nunca existe mudança de status confirmada sem a notificação correspondente registrada, nem o contrário | [09:40] Bruno |
| PRD-NFR-003 | **Isolamento:** um cliente lento ou fora do ar não pode atrasar nem impedir a mudança de status de pedidos | [09:04] Bruno |
| PRD-NFR-004 | **Autenticidade e integridade:** toda mensagem é assinada com HMAC-SHA256 e uma secret exclusiva do webhook | [09:22] Sofia |
| PRD-NFR-005 | **Transporte seguro:** só são aceitas URLs HTTPS | [09:23] Sofia |
| PRD-NFR-006 | **Tamanho:** mensagens acima de 64KB não são enviadas (erro, sem truncar) | [09:24] Larissa |
| PRD-NFR-007 | **Entrega pelo menos uma vez:** o cliente pode receber duplicatas e deve deduplicar pelo identificador do evento | [09:26] Larissa |
| PRD-NFR-008 | **Tempo de resposta do cliente:** sem resposta em 10s conta como falha e entra no retry | [09:42] Diego |
| PRD-NFR-009 | **Auditoria:** todo reenvio manual registra o autor | [09:36] Sofia |
| PRD-NFR-010 | **Sem infraestrutura nova:** a solução usa o banco e a stack existentes | [09:07] Diego |
| PRD-NFR-011 | **Revisão de segurança** de pelo menos 2 dias úteis antes do deploy | [09:46] Sofia |

## 8. Decisões e trade-offs principais

| Decisão | Trade-off aceito | ADR |
|---|---|---|
| Notificação assíncrona via outbox no MySQL, em vez de chamada direta | Até ~2s de atraso em troca de não travar pedidos nem perder eventos | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| Worker separado, consulta a cada 2s, instância única | Simplicidade em troca de escala limitada e ordem garantida só por pedido no caminho feliz | [ADR-002](adrs/ADR-002-worker-separado-em-polling.md) |
| 5 retentativas em ~15h e depois DLQ | Entregas tardias e reenvio manual em troca de não perder eventos nem tentar para sempre | [ADR-003](adrs/ADR-003-retry-backoff-e-dlq.md) |
| HMAC com secret por webhook e rotação | Secret guardada em formato recuperável em troca de padrão de mercado e isolamento entre clientes | [ADR-004](adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md) |
| At-least-once em vez de exactly-once | O cliente precisa deduplicar | [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| Mensagem enxuta, sem itens, congelada no momento da mudança | O cliente consulta `GET /orders/:id` para ter detalhes | [ADR-007](adrs/ADR-007-payload-snapshot-na-insercao.md) |

## 9. Dependências

| Dependência | Tipo | Origem |
|---|---|---|
| Revisão de segurança da Sofia (≥ 2 dias úteis) antes do deploy | Pessoa / agenda | [09:46] Sofia |
| Documentação no portal do desenvolvedor (deduplicação, assinatura, integração via API) | PM | [09:26] Marcos, [09:40] Marcos |
| Confirmação do prazo com a Atlas | PM | [09:47] Marcos |
| Usuários do OMS que representam cada cliente, para cadastrar os webhooks | Operação | [09:32] Marcos |
| Clientes implementando um endpoint HTTPS, validação de assinatura e deduplicação | Externa | [09:20] Sofia, [09:25] Diego |
| Implantação e supervisão de um novo processo (worker) | Operação | [09:11] Diego |

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação | Origem |
|---|---|---|---|---|---|
| PRD-RSK-01 | **Atraso na entrega e churn da Atlas** | Média | Alto | Escopo fechado (e-mail, dashboard e rate limit fora); estimativa de 3 sprints com revisão já reservada; Marcos alinha o prazo com a Atlas | [09:00], [09:46]–[09:47] |
| PRD-RSK-02 | **Cliente processa a mesma notificação duas vezes** | Média | Médio | Identificador único por evento; documentação destacada no portal | [09:25] Sofia, [09:26] Marcos |
| PRD-RSK-03 | **Vazamento de secret de um cliente** | Baixa | Alto | Secret exclusiva por webhook (o vazamento não afeta outros) e rotação com 24h de convivência | [09:21]–[09:22] |
| PRD-RSK-04 | **Muitas notificações em sequência sobrecarregam o cliente** (ex.: 50 em um minuto) | Média | Médio | Monitorar o volume por cliente e decidir rate limiting depois | [09:38] Diego, [09:39] Larissa |
| PRD-RSK-05 | **Notificações atrasam** porque o worker parou ou um cliente lento segura a fila | Média | Alto | Alerta de fila parada; timeout de 10s por cliente | [09:42] Diego; [FDD §12](FDD.md#12-observabilidade) |
| PRD-RSK-06 | **Cliente não percebe que o webhook está falhando** (sem aviso proativo nesta fase) | Média | Médio | Histórico de entregas consultável; e-mail de alerta na próxima fase | [09:34], [09:37] |
| PRD-RSK-07 | **A URL cadastrada é usada para alcançar sistemas internos da empresa** (SSRF) | Média | Crítico | Bloquear endereços internos no cadastro e no envio ([FDD §11.1](FDD.md#111-proteção-contra-ssrf-severidade-crítica)). Levantado na revisão; não discutido na reunião | [09:19] Sofia |
| PRD-RSK-08 | **Um cliente vê ou altera os webhooks de outro cliente** | Média | Alto | Liberado para qualquer usuário autenticado "por enquanto". A revisão recomenda o isolamento como pré-condição antes de abrir a API a usuários dos clientes ([FDD §11.3](FDD.md#113-isolamento-entre-clientes)) | [09:37] Sofia |

## 11. Critérios de aceitação

| ID | Critério (Dado / Quando / Então) | Origem |
|---|---|---|
| PRD-AC-001 | **Dado** um webhook da Atlas assinando `SHIPPED`, **quando** um pedido da Atlas vai para `SHIPPED`, **então** o endpoint da Atlas recebe a notificação assinada em menos de 10s | PRD-FR-001, PRD-NFR-001 |
| PRD-AC-002 | **Dado** um webhook assinando só `DELIVERED`, **quando** o pedido vai para `SHIPPED`, **então** nada é enviado para ele | PRD-FR-006 |
| PRD-AC-003 | **Quando** o usuário cadastra um webhook, **então** recebe a secret na resposta, e ela não aparece em nenhuma outra consulta além da rotação | PRD-FR-002 |
| PRD-AC-004 | **Quando** o usuário informa uma URL `http://`, **então** o cadastro é recusado com erro de validação | PRD-NFR-005 |
| PRD-AC-005 | **Dado** um endpoint fora do ar por 2h, **quando** ele volta, **então** a notificação é entregue numa das retentativas, sem intervenção | PRD-FR-009 |
| PRD-AC-006 | **Dado** um endpoint fora do ar por mais de 15h, **então** o evento aparece na DLQ e um ADMIN consegue reenviá-lo; um OPERATOR não consegue | PRD-FR-010, PRD-FR-011 |
| PRD-AC-007 | **Quando** a secret é rotacionada, **então** mensagens assinadas validam com a secret antiga por 24h e só com a nova depois disso | PRD-FR-007 |
| PRD-AC-008 | **Quando** a mesma notificação é reenviada, **então** ela carrega o mesmo identificador de evento | PRD-FR-012 |
| PRD-AC-009 | **Quando** o usuário consulta o histórico, **então** vê até 100 entregas com resultado, status HTTP, tempo de resposta e payload | PRD-FR-008 |
| PRD-AC-010 | **Dado** um cliente fora do ar, **quando** o operador muda status de pedidos, **então** a operação responde normalmente | PRD-NFR-003 |

Os critérios técnicos detalhados estão no [FDD §16](FDD.md#16-critérios-de-aceite-técnicos).

## 12. Estratégia de testes e validação

| Etapa | O que valida | Responsável |
|---|---|---|
| Testes automatizados (unitário, API, worker) | Critérios PRD-AC-001..010 e FDD-AC-01..16 | Bruno, Diego |
| Revisão de segurança (≥ 2 dias úteis) | Geração de secret, HMAC, permissões do replay, ausência de secret em logs | Sofia ([09:46]) |
| Homologação ponta a ponta com receptor de teste | Latência < 10s, assinatura, retries, DLQ e replay | Time de engenharia |
| Piloto com Atlas, MaxDistribuição e Nova Cargo | Integração real; validação de assinatura e deduplicação do lado deles | Marcos |
| Acompanhamento pós-lançamento (30 dias) | PRD-OBJ-01, 04 e 05; volume por cliente para decidir rate limiting; volume de falhas para decidir o e-mail de alerta | Marcos, Larissa |
