# Questões em Aberto

Consolida tudo que ainda precisa ser **decidido** ou **confirmado** (os "A CONFIRMAR" espalhados pelos documentos). Prioridade **P1** = bloqueia/define o MVP (resolver na Semana 1); **P2** = importante mas não bloqueia; **P3** = fase futura.

## Integrações — APIs de afiliados (P1, resolver na Semana 1)

| # | Questão | Onde impacta |
|---|---------|--------------|
| Q-01 | **Shopee BR:** endpoint GraphQL exato da Open API de afiliados, método de assinatura e escopos. | `docs/05` |
| Q-02 | **Shopee:** `productOfferV2` retorna **preço por variação**? Campos exatos na versão BR. | `docs/05`, `docs/04` |
| Q-03 | **Shopee:** suporta **sub_id/sub-tags** para separar origem (canal/site)? | `docs/05`, `docs/07` |
| Q-04 | **Shopee:** cadastro **sem exigir vendas**? (premissa atual) | `docs/05` |
| Q-05 | **AliExpress:** método oficial de **detalhe de produto por ID** e se traz preço por variação. | `docs/05` |
| Q-06 | **AliExpress:** limites de **source_values/sub-id** por chamada. | `docs/05` |
| Q-07 | **AliExpress:** cadastro **sem exigir vendas**? (premissa atual) | `docs/05` |
| Q-08 | **Rate limits** oficiais de Shopee e AliExpress (para o orçamento de chamadas). | `docs/03`, `docs/05` |

## Conformidade (P1)

| # | Questão | Onde impacta |
|---|---------|--------------|
| Q-09 | **Redirecionamento/cloaking:** Shopee e AliExpress permitem redirecionador no domínio próprio? | `docs/08`, `docs/03` |
| Q-10 | **Aviso de afiliado:** texto/posição exigidos por cada programa. | `docs/08`, `docs/06`, `docs/07` |
| Q-11 | **Exibição de preço em site:** frequência mínima de atualização exigida e regras sobre exibir/guardar **histórico**. | `docs/08`, `docs/07` |
| Q-12 | **Marcas/categorias restritas** em Shopee/AliExpress (relevante para corrida). | `docs/08`, `docs/05` |
| Q-13 | **LGPD:** base legal, necessidade de `ip_hash`/`ua_hash` nos cliques, e forma do canal de solicitação do titular (e-mail?). | `docs/08`, `docs/04` |
| Q-14 | **Revisão jurídica** dos termos de uso e da política de privacidade. | `docs/08` |

## Produto / parâmetros do motor (P2)

| # | Questão | Onde impacta |
|---|---------|--------------|
| Q-15 | Método final do **preço de referência** (mediana 30d vs mínimo estável 90d) — validar com dados reais. | `docs/05` |
| Q-16 | **Margem em camadas** por categoria (premissa: Boa oferta 10–20%, Queda forte ≥20%, mínimo R$15) — validar na Semana 4. Ver ADR 0011. | `docs/05` |
| Q-17 | **Cooldown** de republicação (premissa: 7 dias) e regra anti-flap (premissa: 2 verificações). | `docs/05` |
| Q-18 | **Cupom (ADR 0010):** as APIs de Shopee/AliExpress expõem **validade e preço do cupom** (para testar antes de postar)? Aplicar cupom **cancela a atribuição da comissão**? | `docs/05`, `docs/08` |
| Q-19 | Quanto tempo de histórico é suficiente para as **primeiras ofertas confiáveis** (maturação na Semana 4). | `docs/09` |

## Infraestrutura (P2)

| # | Questão | Onde impacta |
|---|---------|--------------|
| Q-20 | **Domínio** final do projeto (usado em site, redirecionador, textos). | `docs/03`, `docs/06`, `docs/07` |
| Q-21 | **Reverse proxy:** Caddy vs nginx+certbot. | `docs/03` |
| Q-22 | ~~Site lê banco direto ou via API dedicada?~~ **RESOLVIDO (ADR 0001):** site consome **API FastAPI** dedicada. | `docs/03`, `docs/07` |
| Q-23 | **Logs/monitoramento:** arquivos rotacionados vs stack (Loki/Grafana). | `docs/03` |
| Q-24 | **Analytics** do site com foco em privacidade (Plausible/Umami self-hosted?). | `docs/07`, `docs/08` |
| Q-25 | **Deploy:** CI/CD vs manual no MVP. | `docs/03` |

## Métricas / negócio (P2)

| # | Questão | Onde impacta |
|---|---------|--------------|
| Q-26 | **Metas numéricas** do critério de 60 dias (X inscritos, Y comissões) — definir na Semana 4 com base no tamanho das comunidades. | `docs/09` |
| Q-27 | **Granularidade de sub-id** por programa para atribuir vendas por origem. | `docs/07`, `docs/05` |

## Fases futuras (P3)

| # | Questão | Onde impacta |
|---|---------|--------------|
| Q-28 | **WhatsApp Cloud API:** preços atuais de template/conversa e se um alerta opt-in se enquadra como "utilidade". | `docs/decisoes/0002`, `docs/09` |
| Q-29 | **Amazon Creators API:** número exato de vendas qualificadas, regras e URL oficial da doc. | `docs/05` |
| Q-30 | **Mercado Livre:** surgiu API oficial de afiliados? | `docs/05` |
| Q-31 | **Feeds automáticos** de catálogo (quais endpoints de oferta cada programa expõe e limites). | `docs/decisoes/0009`, `docs/05` |
| Q-32 | **Limite de produtos por usuário** na fase 2 (premissa: 50) — validar com carga real. | `docs/02`, `docs/06` |

---

## Premissas assumidas (revisar quando possível)

- Público de corrida tem comunidades ativas e acessíveis para o lançamento (`docs/01`).
- Shopee e AliExpress liberam API de afiliados sem exigir vendas (`docs/05`).
- Preço de referência = mediana dos últimos 30 dias (`docs/05`).
- Um único VPS (~4 vCPU / 8 GB) atende o MVP (`docs/03`).
- É possível maturar histórico suficiente em poucos dias para as primeiras ofertas (`docs/09`).
- Volume de posts vem de catálogo ~800–1.500 produtos (lançando com ~200–400); a curadoria manual escala até virar gargalo (`docs/09`, ADR 0011).
