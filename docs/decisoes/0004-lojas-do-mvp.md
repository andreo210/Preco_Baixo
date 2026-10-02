# 0004 — Shopee + AliExpress no MVP; Amazon e Mercado Livre depois

**Status:** aceita · **Data:** 2026-09-30

## Contexto
Precisamos de lojas com **API oficial de afiliados** que **não exija vendas** para liberar acesso, já que começamos do zero.

## Decisão
MVP com **Shopee** e **AliExpress**. Amazon e Mercado Livre em fases futuras.

## Alternativas / razões
- **Amazon:** PA-API 5.0 **descontinuada em 15/05/2026**; a **Creators API** exige **vendas qualificadas** (~10/30 dias, A CONFIRMAR) para acesso → inviável sem vendas prévias. Fase futura.
- **Mercado Livre:** **não há API oficial de afiliados** (geração de link é manual pelo painel). Sem caminho oficial automatizável, e não fazemos scraping onde proibido. Fase futura.
- **Shopee/AliExpress:** têm API de afiliados e, por premissa, liberam acesso sem exigir vendas. Fortes justamente no nicho de corrida (acessórios/gadgets).

## Consequências
- MVP cobre duas lojas com bom volume no nicho.
- Várias regras específicas ficam **A CONFIRMAR** na Semana 1 (endpoints BR, sub-id, rate limits, marcas restritas) — ver `docs/05` e `docs/questoes-em-aberto.md`.
- Amazon/ML entram via novos adaptadores (ADR 0003) quando houver caminho.
