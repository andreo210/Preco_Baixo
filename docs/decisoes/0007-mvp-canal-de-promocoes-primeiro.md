# 0007 — MVP = canal de promoções; rastreador pessoal na fase 2

**Status:** aceita · **Data:** 2026-09-30

## Contexto
Duas formas de entregar valor rodam no **mesmo motor** (coleta/histórico/detecção): um **canal de promoções** (broadcast, um-para-muitos) e um **rastreador pessoal** (um-para-um, o usuário escolhe o produto). Precisávamos escolher por onde começar.

## Decisão
MVP = **canal público de promoções** no Telegram (+ site). O **rastreador pessoal por DM** fica para a **fase 2**.

## Alternativas
- **Rastreador pessoal primeiro:** mais diferenciado e com mais retenção, porém mais complexo (estado por usuário, limite de produtos, push 1-a-1 com controle de taxa) e sem audiência inicial.
- **Os dois no MVP:** dobra a superfície de bot/mensagens e aperta o prazo de 4 semanas.

## Consequências
- MVP mais simples: sem conta de usuário, sem limite por usuário, sem push 1-a-1.
- Monetiza antes (cada post atinge toda a audiência) e constrói público próprio.
- Enterra, no MVP, o problema de custo por mensagem (broadcast no Telegram é grátis).
- Diferencial: "só quedas reais verificadas no histórico" (vs canais que só repostam).
- Rastreador pessoal e WhatsApp entram depois via adaptadores (ADR 0003).
