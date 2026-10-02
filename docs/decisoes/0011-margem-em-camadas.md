# 0011 — Margem em camadas; volume via catálogo, não via régua mais baixa

**Status:** aceita · **Data:** 2026-09-30

## Contexto
Postar só quando o preço cai contra o histórico pode gerar **poucos posts** e deixar o canal mudo. Precisávamos de volume **sem** enfraquecer o diferencial ("queda verificada"). O gatilho já é uma **margem** contra o preço de referência (não é "menor preço da história").

## Decisão
1. A queda é medida contra o **preço de referência (base)**, com mínimo de **≥ R$ 15**, e classificada em **dois níveis**:
   - **✅ Boa oferta:** 10% a 20%.
   - **🔥 Queda forte:** ≥ 20%.
2. O **volume** de posts vem de **ampliar o catálogo** (meta ~800–1.500 produtos; lançar com ~200–400), **não** de baixar a margem.
3. **Não** usar o **desconto anunciado pela loja** (âncora "De/Por") como gatilho — é a âncora inflada que combatemos.
4. Pisos e mínimo são **configuráveis por categoria**, ajustados na Semana 4 com dados reais.

## Alternativas
- **Baixar a margem mínima** (ex.: 5%): mais posts, porém ofertas fracas que cansam o inscrito.
- **Usar o desconto anunciado pela loja:** mais volume, mas reintroduz a queda falsa (contra o diferencial).

## Consequências
- Dois níveis dão volume (o tier "Boa oferta" enche o canal) mantendo o selo "Queda forte" significativo.
- O principal esforço operacional passa a ser **curadoria de catálogo**; se virar gargalo, antecipar feeds automáticos (fase 2) — ver ADR 0009.
- Campo `Publicacao.nivel` no modelo (ver `docs/04`); formato do post em `docs/06`.
