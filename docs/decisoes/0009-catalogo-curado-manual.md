# 0009 — Catálogo curado manualmente no MVP; feeds automáticos depois

**Status:** aceita · **Data:** 2026-09-30

## Contexto
No canal-first (ADR 0007), os produtos não vêm dos usuários. Precisávamos definir a **fonte do catálogo** que o motor vigia. O diferencial ("queda verificada no histórico") exige **já estar acompanhando** o produto antes de anunciar uma queda.

## Decisão
No MVP, o **admin adiciona produtos manualmente** por link (comando `/add`), começando com ~200–400 produtos de corrida e crescendo para ~800–1.500 conforme a necessidade de volume (ver ADR 0011). **Feeds automáticos** (Shopee `productOfferV2`, hot products AliExpress) ficam para fase futura — e são antecipados se a curadoria manual virar gargalo.

## Alternativas
- **Feeds automáticos desde o MVP:** escala sem trabalho manual, mas (a) **cold-start**: produto recém-ingerido não tem histórico para provar queda; (b) integrar feeds de duas APIs é trabalho extra em 4 semanas; (c) catálogo gigante dilui o histórico e estoura rate limits.

## Consequências
- Histórico confiável mais rápido num conjunto focado → quedas críveis desde o lançamento.
- Menos a construir no MVP; respeita rate limits e o VPS.
- "Manual" = escolher **quais** produtos entram (o motor coleta o preço sozinho); não é digitar preço, e não é atendimento.
- Dá para ligar feeds na fase 2/3 quando soubermos quais categorias convertem.
