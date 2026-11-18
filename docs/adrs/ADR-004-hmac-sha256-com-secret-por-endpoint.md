# ADR-004 — Autenticação de webhooks com HMAC-SHA256 e secret por endpoint

| Campo | Valor |
|---|---|
| **Status** | Aceito, com detalhes de rotação **pendentes da revisão de segurança** (ver "Pontos pendentes") |
| **Data** | 2026-11-16 (decisão tomada na reunião técnica; ver [TRANSCRICAO.md](../../TRANSCRICAO.md) [09:22]) |
| **Decisores** | Sofia (Segurança), Larissa (Tech Lead) |
| **Consultados** | Bruno, Diego, Marcos |
| **Relacionados** | [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) |

## Contexto

Os webhooks enviam dados de pedidos para endpoints **fora da nossa infraestrutura** ([09:19] Sofia). O cliente precisa verificar duas coisas:

1. que a requisição **veio de nós** (autenticidade);
2. que o payload **não foi adulterado** (integridade).

Já houve cliente que vazou secret em log de aplicação ([09:22] Diego), então a solução precisa limitar o raio de impacto de um vazamento e permitir troca da secret.

No código atual, o único segredo de usuário guardado é a senha, e com hash irreversível (`bcrypt`, em [src/modules/auth/auth.service.ts](../../src/modules/auth/auth.service.ts)). O logger só faz redact de `password`, `passwordHash`, `token` e `accessToken` ([src/shared/logger/index.ts](../../src/shared/logger/index.ts)).

## Problema

Como o cliente autentica as chamadas que recebe, de forma padronizada, com isolamento entre clientes e suporte a rotação?

## Decisão

1. **HMAC-SHA256 sobre o corpo da requisição**, com a assinatura no header **`X-Signature`** ([09:20] Sofia, [09:20] Sofia, fechado em [09:22] Sofia).
2. **Uma secret única por endpoint de webhook**, nunca global ([09:21] Sofia). A secret é **gerada pela plataforma** e **devolvida na criação** ([09:31] Marcos).
3. A configuração do webhook guarda `url`, `secret`, `customer_id` e o estado ativo ([09:21] Bruno, confirmado por [09:21] Sofia).
4. **Rotação via API:** o cliente pede uma nova secret, e a antiga **continua válida por 24 horas** em paralelo. Depois disso, deixa de valer ([09:21] Sofia).
5. **TLS obrigatório:** a URL do webhook precisa ser `https`, e `http` é recusado com erro de validação no schema Zod ([09:23] Sofia).
6. **Revisão de segurança dedicada:** pelo menos 2 dias úteis antes do deploy, com foco em HMAC e geração de secret ([09:46] Sofia).

### Detalhamento técnico (proposta deste pacote, sujeita à revisão da Sofia)

- **Formato do header:** `X-Signature: sha256=<hex>`, calculado sobre os **bytes exatos** do corpo enviado.
- **Geração:** `crypto.randomBytes(32)` (módulo nativo do Node, sem dependência nova).
- **Exposição:** a secret só aparece na resposta de **criação** e de **rotação**. Listagens e edições nunca a devolvem.
- **Armazenamento:** a secret precisa ser **recuperável** para calcular o HMAC. Ela **não pode** receber hash com bcrypt, ao contrário de `passwordHash`. Por isso é gravada **cifrada** (proposta, ver "Pontos pendentes") e incluída nos caminhos de `redact` do Pino.

### Pontos pendentes (questões em aberto no [RFC](../RFC.md#7-questões-em-aberto))

- **Assinatura durante o grace period de 24h:** a reunião definiu que a secret antiga "fica válida", mas não definiu **com qual secret o servidor assina** nesse intervalo. Proposta: enviar as duas assinaturas, `X-Signature: sha256=<hmac_nova>,sha256=<hmac_antiga>`, e o cliente aceita se qualquer uma conferir.
- **Criptografia da secret em repouso:** não foi discutida na reunião. A revisão humana propõe AES-256-GCM com chave fora do banco ([FDD §11.2](../FDD.md#112-secret-em-repouso)), pendente de aprovação da Sofia.
- **Validação de destino (SSRF):** a assinatura protege o cliente, mas não protege o OMS de uma URL que aponte para a rede interna. Exigir HTTPS não basta. A revisão humana propõe validação no cadastro e no envio, com IP pinning ([FDD §11.1](../FDD.md#111-proteção-contra-ssrf-severidade-crítica)). Severidade crítica.
- **Auditoria da rotação:** log `webhook_secret_rotated` com `userId` ([FDD §11.4](../FDD.md#114-auditoria-de-operações)).
- **`X-Timestamp` fora da assinatura:** como a decisão é assinar só o corpo, o `X-Timestamp` ([09:44] Diego) não é coberto pelo HMAC. A proteção contra replay depende do `timestamp` que vai dentro do payload, que é assinado ([ADR-007](ADR-007-payload-snapshot-na-insercao.md)).

## Justificativa

- HMAC-SHA256 é o padrão de mercado e "todo cliente sério tem biblioteca pra isso" ([09:20] Sofia).
- A secret por endpoint isola os clientes: "se vaza uma, vaza tudo" no modelo global ([09:21] Sofia).
- A rotação com sobreposição de 24h deixa o cliente migrar sem perder eventos ([09:21] Sofia).

## Alternativas consideradas

| Alternativa | Origem | Motivo do descarte |
|---|---|---|
| **Secret global da plataforma** | [09:21] Sofia | Um único vazamento comprometeria todos os clientes ([09:21] Sofia). |
| **Depender só de TLS, sem assinatura** | Plausível, não discutida | O TLS autentica o servidor do cliente, mas não prova ao cliente que a chamada veio do OMS, e qualquer um que conheça a URL pode chamá-la. Não atende o requisito de [09:19] Sofia. |
| **Assinatura assimétrica (chave pública/privada) ou mTLS** | Plausível, não discutida | Mais complexa para o cliente integrar e foge do "padrão de mercado" citado. Fica como evolução, se houver necessidade. |

## Consequências

### Positivas

- O cliente valida autenticidade e integridade com bibliotecas padrão.
- O vazamento de uma secret afeta um único endpoint.
- A rotação acontece sem janela de indisponibilidade.
- Nenhuma dependência nova (`node:crypto`).

### Negativas

- **A secret fica em formato recuperável no banco:** acesso de leitura ao banco expõe as secrets. A mitigação depende da decisão sobre criptografia em repouso (pendente).
- **Complexidade no cliente** durante a rotação, se forem enviadas duas assinaturas.
- **Mais atenção nos logs:** é preciso estender o `redact` do Pino, porque um log descuidado com o objeto do webhook vazaria a secret.
- A responsabilidade pela validação fica com o cliente. Se ele não validar, a assinatura não protege nada.

### Trade-off explícito

Escolhemos **simplicidade e padrão de mercado** (segredo compartilhado simétrico) em vez de **não-repúdio e ausência de segredo armazenado** (assinatura assimétrica). Em troca, aceitamos guardar secrets recuperáveis e cuidar da rotação.

## Referências

- [TRANSCRICAO.md](../../TRANSCRICAO.md): [09:19]–[09:23], [09:31], [09:44], [09:46]
- [src/shared/logger/index.ts](../../src/shared/logger/index.ts): `redactPaths`
- [src/modules/auth/auth.service.ts](../../src/modules/auth/auth.service.ts): uso de `bcrypt` para senhas (contraste)
- [FDD §8.6 Rotação de secret](../FDD.md#86-post-apiv1webhooksidrotate-secret) e [FDD §11 Segurança](../FDD.md#11-segurança)
