# 05 — Integrações

Uma seção por loja e pelo Telegram: autenticação, recursos usados, limites, geração do link de afiliado, regras do programa e **fontes**. Inclui a definição de **queda de preço real** e da **frequência adaptativa**. Tudo que não foi confirmado em fonte oficial está marcado como **A CONFIRMAR** e repetido em `docs/questoes-em-aberto.md`.

> Regra-mãe: **preferir API oficial a scraping**. Nunca coletar onde os termos proíbem. Se uma loja não oferecer API adequada, ela fica fora até haver caminho oficial.

---

## Definição de "queda de preço real" (vale para todas as lojas)

Uma queda só é publicada se **todas** as condições forem verdadeiras:

1. **Mesma variação.** Comparar sempre o mesmo SKU/tamanho/cor. Nunca comparar o preço de uma variação com o de outra (ex.: tamanho 38 vs 42).
2. **Disponível.** O produto (aquela variação) está em estoque. Preço de item **esgotado** é ignorado.
3. **Preço de referência.** Calculado do histórico da **mesma variação**. **Premissa:** usar a **mediana dos últimos 30 dias** (ou o percentil 50) como "preço normal". Alternativa: menor preço "estável" dos últimos 90 dias. Parametrizável. **A CONFIRMAR** o método final após ver dados reais.
4. **Margem em camadas (tiers).** A queda é medida **contra o preço de referência (base)**, com diferença mínima de **≥ R$ 15** (evita "queda" de centavos), e classificada em dois níveis (ADR 0011):
   - **✅ Boa oferta:** queda de **10% a 20%**.
   - **🔥 Queda forte:** queda **≥ 20%**.
   **Premissa inicial** (ajustável por categoria na Semana 4): pisos de 10% / 20% e mínimo de R$ 15. O **volume** de posts vem de **ampliar o catálogo** (meta ~800–1.500 produtos), **não** de baixar a régua nem de usar como gatilho o **desconto anunciado pela loja** (âncora "De/Por" inflada).
5. **Cupom — comparar maçã com maçã + validar antes de postar (ADR 0010).** O **preço base (sem cupom)** é a **série canônica**: o preço de referência e o **% da manchete** usam sempre **base vs base** (cupom **nunca** entra no %, senão vira queda falsa). Um cupom só aparece como **bônus** no post se for **validado via API oficial no momento da publicação** (confirmando que está ativo e qual o preço final). Mesmo validado, o post traz a **ressalva de validade/limite** ("pode expirar / ter limite"), porque:
   - o cupom pode **expirar/esgotar entre o post e o clique**;
   - cupons **por usuário/condição** (valor mínimo, 1ª compra, app-only) podem não valer para todos — não dá para testar pela ótica do usuário.
   **A CONFIRMAR:** (a) se as APIs de Shopee/AliExpress **expõem validade e preço do cupom** (sem isso não há como testar oficialmente, e não fazemos scraping); (b) se aplicar **código de cupom cancela a atribuição da comissão** de afiliado. Ofertas cujo destaque principal seja o cupom ficam para fase futura.
6. **Frete de fora.** Comparar o **preço do produto**, não o frete (que varia por CEP). Mudança de "frete grátis" pode ser mencionada no post, mas não conta na % de queda.
7. **Cooldown.** Não republicar o mesmo produto por **N dias** (Premissa: 7) nem enquanto `Produto.cooldown_ate` estiver no futuro. Evita spam e re-post da mesma oferta.
8. **Persistência (anti-flap).** A queda deve se manter por pelo menos **2 verificações** antes de publicar, para não postar preço que oscilou por erro momentâneo. **Premissa.**

## Frequência adaptativa (vale para todas)

- Cada loja tem um **orçamento de chamadas** por janela, derivado do seu rate limit oficial (**A CONFIRMAR** por loja).
- Prioridade maior para produtos com **mais cliques recentes**, preço **volátil** e preço **próximo da referência** (candidatos a cair).
- Produtos "frios": intervalo maior.
- Nunca ultrapassar o orçamento da loja. Fase 2: peso extra para produtos acompanhados por mais usuários.

---

## Telegram (canal + bot)

- **Papel no MVP:** publicar ofertas no **canal público** e receber **comandos de admin** pelo bot. Fase 2: DM com usuários (rastreador pessoal).
- **Autenticação:** token do bot obtido no **@BotFather**. O bot precisa ser **administrador do canal** para publicar.
- **Recursos usados:** `sendPhoto`/`sendMessage` com botões inline (`InlineKeyboardMarkup`) para "Comprar" e "Ver histórico"; comandos do bot; (fase 2) webhook ou long polling para DMs.
- **Limites (confirmados, fonte oficial Telegram):**
  - ~**1 mensagem/segundo** para um mesmo chat.
  - Até **20 mensagens/minuto** para um mesmo **grupo**.
  - ~**30 mensagens/segundo** no total (global). Acima disso só com **Paid Broadcasts** (até 1000/s, custando **0,1 Star** por mensagem acima de 30/s).
  - O Telegram afirma que os números são **dinâmicos**; tratar como teto conservador.
- **Impacto no projeto:** postar num canal é baixo volume (poucas ofertas/dia), bem dentro dos limites. O cuidado maior aparece na **fase 2** (push 1-a-1 para muitos usuários).
- **Fontes:** [Telegram Bots FAQ — broadcasting limits](https://core.telegram.org/bots/faq) · [Bot API reference](https://core.telegram.org/bots/api)

---

## Shopee (Programa de Afiliados / Open API)

- **Papel:** loja do MVP. Buscar dados do produto e gerar link de afiliado; coletar preço periodicamente.
- **Autenticação:** **App ID** e **API Key** obtidos no painel do afiliado (seção "Open API"). No **Brasil**, a Open API de afiliados é exposta como **GraphQL**. **A CONFIRMAR:** endpoint GraphQL BR exato, método de assinatura das requisições e escopos.
- **Recursos (pelo que a doc oficial indica):**
  - `productOfferV2` — dados de oferta/produto de afiliado (para obter preço e metadados).
  - `generateShortLink` — geração do link curto de afiliado.
  - Relatórios de conversão/cliques/comissão.
  - **A CONFIRMAR:** nomes/campos exatos na versão BR (GraphQL) e se `productOfferV2` retorna preço por **variação**.
- **Geração do link de afiliado:** via `generateShortLink` a partir da URL do produto, associando o **sub_id** de origem (canal/site) para rastrear a venda. **A CONFIRMAR:** suporte a sub_id/sub-tags na Shopee BR.
- **Limites (rate):** **A CONFIRMAR** — não confirmados em fonte oficial.
- **Regras do programa (exibição/armazenamento de preço, redirecionamento, aviso de afiliado):** **A CONFIRMAR** nos termos do programa BR.
- **Cadastro sem exigir vendas:** **Premissa** (motivo de escolher Shopee no MVP) — **A CONFIRMAR** nos termos atuais.
- **Fontes:** [Shopee Affiliate — API Access (Help Center)](https://help.shopee.sg/portal/10/article/191702-API-Access) · [Shopee Open Platform](https://open.shopee.com/) *(A CONFIRMAR o portal específico de afiliados BR)*

---

## AliExpress (Affiliate / Open Platform — "Portals")

- **Papel:** loja do MVP. Buscar dados do produto e gerar link de afiliado; coletar preço.
- **Autenticação:** **App Key** + **App Secret** + **Tracking ID**, criados no **AliExpress Portals** / Open Platform. Requisições assinadas (`sign_method`).
- **Recursos:**
  - `aliexpress.affiliate.link.generate` — gera o link de afiliado (parâmetros incluem `app_key`, `timestamp`, `tracking_id`, `promotion_link_type`, `source_values`, `sign_method`, `v`).
  - Endpoint: `https://api-sg.aliexpress.com/sync`.
  - Métodos de detalhe de produto / hot products (para preço). **A CONFIRMAR:** método exato de detalhe de produto por ID e se retorna preço por variação.
- **Geração do link de afiliado:** `aliexpress.affiliate.link.generate` com `tracking_id`; usar `source_values` para diferenciar origem (canal/site). **A CONFIRMAR:** limites de sub-id/source por chamada.
- **Limites (rate):** **A CONFIRMAR** — não confirmados em fonte oficial.
- **Regras do programa (exibição de preço em site/canal, redirecionamento, aviso):** **A CONFIRMAR** nos termos do AliExpress Affiliate.
- **Cadastro sem exigir vendas:** **Premissa** — **A CONFIRMAR**.
- **Fontes:** [AliExpress Open Platform — Getting Started](https://openservice.aliexpress.com/doc/doc.htm) · [API Reference](https://openservice.aliexpress.com/doc/api.htm) · [AliExpress Portals](https://portals.aliexpress.com/)

---

## Amazon (fase futura)

- **Situação (importante):** a **Product Advertising API 5.0 (PA-API) foi descontinuada em 15/05/2026**. O caminho atual é a **Amazon Creators API**.
- **Requisito de acesso:** exige **vendas qualificadas** para obter e **manter** o acesso (relatado: ~**10 vendas qualificadas em 30 dias**; regras já foram 3-em-180-dias e mudaram). Por isso Amazon **não entra no MVP** — precisamos primeiro gerar vendas por outros meios. **A CONFIRMAR:** número exato e regras atuais da Creators API.
- **Geração de link / recursos:** **A CONFIRMAR** na doc da Creators API.
- **Fontes:** [Product Advertising API 5.0 — docs](https://webservices.amazon.com/paapi5/documentation/) · [API Rates (PA-API)](https://webservices.amazon.com/paapi5/documentation/troubleshooting/api-rates.html) · **A CONFIRMAR:** URL oficial da Creators API.

---

## Mercado Livre (fase futura)

- **Situação:** **não há API oficial e documentada do programa de afiliados.** A geração de links de afiliado é **manual, pelo painel**. Existe a API geral de desenvolvedores do Mercado Livre, mas **não** para afiliados. **A CONFIRMAR:** se surgiu endpoint oficial de afiliados.
- **Consequência:** sem API oficial de afiliados, o ML só entra se: (a) a Meli lançar API oficial, ou (b) houver processo manual viável que **não** viole os termos (nunca scraping onde proibido). Fica em fase futura.
- **Fontes:** [Mercado Livre Developers](https://developers.mercadolivre.com.br/pt_br/api-docs-pt-br-1) · [Termos do Programa de Afiliados e Criadores](https://www.mercadolivre.com.br/ajuda/30228)

---

## Resumo de prontidão por loja

| Loja | API oficial de afiliado | Exige vendas p/ acesso | No MVP? |
|------|------------------------|------------------------|---------|
| Shopee | Sim (Open API / GraphQL BR) — detalhes A CONFIRMAR | Premissa: não | **Sim** |
| AliExpress | Sim (Portals / Open Platform) | Premissa: não | **Sim** |
| Amazon | Creators API (PA-API descontinuada 05/2026) | Sim (~10 vendas/30d, A CONFIRMAR) | Não (fase futura) |
| Mercado Livre | Não há API oficial de afiliados | — | Não (fase futura) |

> Os itens "A CONFIRMAR" de integrações devem ser resolvidos na **Semana 1** (cadastro nos programas), pois definem o que é viável no MVP. Ver `docs/09` e `docs/questoes-em-aberto.md`.
