# 07 — Dicionário de dados

Referência de consulta campo a campo. Para o schema visual e as decisões de mapeamento, ver [03 — DER](03-der.md); para o porquê de cada regra de negócio, ver [`../04-modelo-de-dados.md`](../04-modelo-de-dados.md) e [`../05-integracoes.md`](../05-integracoes.md).

## Convenções (valem para todas as tabelas)

- **Dinheiro:** sempre `int` em **centavos**, nunca `float`/`numeric`. Elimina erro de arredondamento na comparação de queda.
- **Datas:** `timestamptz` em UTC. `date` só onde a granularidade é o dia (`PrecoDiario.dia`).
- **Auditoria:** `criado_em` (e `atualizado_em` onde a linha muda) presentes por padrão; descritos uma vez aqui, não repetidos por tabela.
- **Soft-delete:** produto "removido" é `ativo = false`; **histórico nunca é apagado** (é ativo estratégico).
- **Enums** guardados como `text` validado na aplicação (não tipo `enum` do PG) — plugabilidade.
- **Segredos** (tokens/keys) **não** ficam no banco; só referências em `Loja.config`. Os segredos vivem em variáveis de ambiente (RNF-09).

---

## Loja

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | serial | não | PK | | Chave técnica. |
| nome | text | não | | "Shopee", "AliExpress" | Nome de exibição. |
| slug | text | não | UK | "shopee", "aliexpress" | Identificador estável usado no código do adaptador; único. |
| ativo | bool | não | | | Desliga a loja sem apagar dados (falha isolada não derruba as outras). |
| config | jsonb | não | | tracking id, limites de rate, **referências** a segredos | Nunca o segredo em si. |
| criado_em | timestamptz | não | | | Auditoria. |

## Produto

Chave lógica única: **(loja_id, id_externo, variacao_externa)** — cada variação é um produto; nunca se compara preço entre variações.

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | serial | não | PK | | Chave técnica. |
| loja_id | int | não | FK → Loja | | |
| id_externo | text | não | UK(composta) | id do produto na loja | Parte da chave lógica. |
| variacao_externa | text | **sim** | UK(composta) | SKU/tamanho/cor | Parte da chave lógica; nulo quando a loja não distingue variação. Ver tratamento de NULL na UK em [03 — DER](03-der.md). |
| url_canonica | text | não | | | Link limpo do produto. |
| titulo | text | não | | | |
| categoria | text | não | | tenis, relogio-gps, fone, roupa, acessorio | Nicho corrida no MVP. |
| imagem_url | text | sim | | | Usada no post. |
| preco_atual_centavos | int | sim | | ≥ 0 | **Cache** do último preço; verdade é `PrecoHistorico`. Nulo até a 1ª coleta. |
| disponivel | bool | não | | | Último estado de estoque; `false` exclui do cálculo de queda. |
| preco_referencia_centavos | int | sim | | ≥ 0 | **Cache** da referência (mediana ~30d); recalcular quando o histórico muda. |
| ativo | bool | não | | | `false` = pausado **ou** removido (ver zona cinzenta). Não apaga histórico. |
| cooldown_ate | timestamptz | sim | | | Não republicar antes disso; `> agora` ⇒ estado "EmCooldown". |
| prioridade_coleta | smallint | não | | | Peso na frequência adaptativa (cliques/volatilidade). |
| visto_em | timestamptz | não | | | Heartbeat do último check bem-sucedido. |
| criado_em / atualizado_em | timestamptz | não | | | Auditoria. |

## PrecoHistorico

Série canônica de preços. **Insere só quando o preço muda** (+ 1 heartbeat/dia). `preco_centavos` é sempre o **preço base (sem cupom)**.

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | bigserial | não | PK | | Volume alto ⇒ `bigserial`. |
| produto_id | int | não | FK → Produto | | |
| preco_centavos | int | não | | ≥ 0 | **Base, sem cupom** — série canônica; o `%` de queda usa sempre base vs base. |
| disponivel | bool | não | | | Preço de item esgotado não entra no cálculo de queda. |
| cupom_codigo | text | sim | | | Cupom conhecido via API (bônus); **não** entra no cálculo. |
| preco_com_cupom_centavos | int | sim | | ≥ 0 | Preço final com o cupom acima, se conhecido via API. |
| coletado_em | timestamptz | não | | | Momento da leitura; base do particionamento por mês (escala). |

## PrecoDiario (downsample)

Agregado diário por produto — alimenta o gráfico do site e mantém o histórico "para sempre" sem inchar. **Nunca é apagado.**

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| produto_id | int | não | PK(composta) FK → Produto | | |
| dia | date | não | PK(composta) | | 1 linha por produto por dia. |
| preco_min_centavos | int | não | | ≥ 0 | Menor preço base do dia. |
| preco_max_centavos | int | não | | ≥ 0 | Maior preço base do dia. |
| preco_fechamento_centavos | int | não | | ≥ 0 | Último preço base do dia. |

## Publicacao

Cada oferta publicada no canal (alimenta também a página de maiores quedas). Linha imutável após publicada — é registro histórico.

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | serial | não | PK | | |
| produto_id | int | não | FK → Produto | | |
| canal | text | não | | "telegram" (plugável) | |
| preco_no_post_centavos | int | não | | ≥ 0 | Preço base no momento do post. |
| preco_referencia_centavos | int | não | | ≥ 0 | Referência usada; congelada no post. |
| percentual_queda | numeric(5,2) | não | | 0–100 | Calculado **só sobre o preço base**; imutável. |
| nivel | text | não | | `boa_oferta` (10–20%), `queda_forte` (≥20%) | Selo do post (ADR 0011). |
| cupom_codigo | text | sim | | | Exibido como bônus; **não** afeta `percentual_queda`. |
| preco_com_cupom_centavos | int | sim | | ≥ 0 | Preço final com cupom, se houver. |
| cupom_validado_em | timestamptz | sim | | | Quando o cupom foi validado via API antes de publicar (ADR 0010). |
| link_afiliado | text | não | | | Com `sub_id` de origem. |
| status | text | não | | `publicada`, `falha` | |
| mensagem_id | text | sim | | | Id da mensagem no canal; nulo se `status=falha`. |
| publicado_em | timestamptz | não | | | |

## Clique

Clique **anonimizado** no botão "Comprar" (canal ou site). **Não referencia usuário** — nada permite inferir quem acompanha o quê (RF-19, RF-24).

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | bigserial | não | PK | | Volume alto. |
| produto_id | int | não | FK → Produto | | |
| publicacao_id | int | sim | FK → Publicacao | | Nulo se veio de página do site sem post. |
| origem | text | não | | `canal`, `site` | Rastreia de onde veio o clique. |
| sub_id | text | não | | | Sub-ID/tracking enviado ao programa de afiliado. |
| criado_em | timestamptz | não | | | |
| ip_hash | text | sim | | hash salgado | **A CONFIRMAR (LGPD):** só manter se necessário para dedup/antiabuso. |
| ua_hash | text | sim | | hash salgado | Idem. |

---

## Fase 2 (não no MVP)

### Usuario

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | serial | não | PK | | |
| telegram_user_id | bigint | não | UK | | **Único dado pessoal**; exige base legal + política de privacidade. |
| criado_em | timestamptz | não | | | |
| apagado_em | timestamptz | sim | | | Soft-delete para `/apagardados` (LGPD). |

### Acompanhamento

Único: **(usuario_id, produto_id)**. Limite de aplicação: **50 produtos ativos por usuário** (FDM-01).

| Campo | Tipo | Nulo? | Chave | Domínio / valores | Regra / porquê |
|-------|------|-------|-------|-------------------|----------------|
| id | serial | não | PK | | |
| usuario_id | int | não | FK → Usuario | | |
| produto_id | int | não | FK → Produto | | |
| preco_alvo_centavos | int | sim | | ≥ 0 | Alerta se o preço cair abaixo. |
| ativo | bool | não | | | |
| criado_em | timestamptz | não | | | |
