# 0008 — Nicho do MVP: corrida

**Status:** aceita · **Data:** 2026-09-30

## Contexto
Um canal de ofertas genérico compete de frente com Buscapé/Promobit/Zoom e dilui SEO, identidade e curadoria. Um nicho por **comunidade** é mais defensável para um projeto solo. Também precisamos de **fluxo diário de ofertas reais** para o canal não ficar mudo.

## Decisão
Nicho do MVP = **corrida** (tênis, relógios/GPS, fones, roupa técnica, acessórios). Definido por **comunidade**, não por um único produto.

## Alternativas
- **"Qualquer produto":** maior TAM teórico, mas compete com gigantes, dilui SEO e inviabiliza curadoria manual.
- **"Só tênis":** SEO e identidade fáceis, porém (a) fluxo de oferta fino → canal mudo; (b) calçado é o **pior caso** de variação (tamanho/cor) → detecção de queda mais difícil; (c) marca famosa é restrita/réplica em Shopee/AliExpress. Estreito demais.
- **Artigos esportivos em geral:** mais largo, identidade/curadoria mais diluídas.

## Consequências
- SEO viável (long-tail de corrida), identidade clara e lançamento óbvio (grupos de corrida).
- Bom fluxo de ofertas (muito acessório/gadget em Shopee/AliExpress).
- Arquitetura não prende ao nicho: campo `categoria` permite expandir para outros nichos (fase futura) no mesmo motor.
