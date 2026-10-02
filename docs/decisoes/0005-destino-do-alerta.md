# 0005 — "Comprar" direto na loja + "Ver histórico" no site

**Status:** aceita · **Data:** 2026-09-30

## Contexto
O alerta/post pode levar o usuário **direto para a loja** (menos cliques, mais conversão) ou **para a página do produto no site** (histórico, rastreamento, tráfego/SEO). Cada caminho tem um trade-off real.

## Decisão
**Híbrido:** o botão **"Comprar"** vai **direto para a loja** (link de afiliado, via redirecionador conforme regra do programa) e um link **"Ver histórico"** leva à **página do produto no site**.

## Alternativas
- **Só direto na loja:** maximiza conversão, mas não gera tráfego nem prova/SEO no site.
- **Só via site:** ganha tráfego e histórico, mas adiciona um passo e tende a reduzir conversão.

## Consequências
- Não sacrifica conversão (compra é 1 clique) e ainda alimenta o site (tráfego, SEO, credibilidade).
- Rastreamento de origem por **sub-id** (`origem=canal`/`origem=site`) distingue o desempenho de cada caminho.
- O redirecionador próprio só é usado se o programa permitir; senão, link direto do programa (ver `docs/08`).
