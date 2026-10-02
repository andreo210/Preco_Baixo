# 0010 — Cupom: manchete só no preço base; bônus validado antes de postar

**Status:** aceita · **Data:** 2026-09-30

## Contexto
Um cupom pode deixar o preço final baixo. Isso gera dois riscos opostos: (a) **queda falsa** — comparar "com cupom hoje" vs "sem cupom antes" inventa um desconto que não existe no preço real; (b) **economia real ignorada** — o que o comprador paga importa. Cupom também é volátil (expira, tem limite, condições por usuário).

## Decisão
1. O **preço base (sem cupom)** é a **série canônica**: preço de referência e **% da manchete** usam sempre **base vs base**. Cupom **nunca** entra no cálculo do %.
2. Um cupom aparece no post apenas como **bônus**, e **só se validado via API oficial no momento da publicação** (ativo + preço final conhecido).
3. Mesmo validado, o post traz a ressalva **"pode expirar / ter limite"**.
4. Se a API não expõe o cupom, **não** mostramos cupom (não fazemos scraping).
5. Ofertas cujo **destaque principal** seja o cupom ficam para fase futura.

## Alternativas
- **Ignorar cupom totalmente:** simples, mas descarta economia real.
- **Postar cupom como oferta principal:** mais volume/economia, mas arrisca "cupom expirado" no clique (mata a confiança) e pode **cancelar a atribuição da comissão**.

## Consequências
- Protege o diferencial ("queda verificada") e a confiança do canal.
- Validar antes de postar reduz o caso "cupom já morto"; ressalva cobre o resto (expira entre post e clique, condições por usuário).
- Campos no modelo: `PrecoHistorico.cupom_codigo/preco_com_cupom_centavos`; `Publicacao.cupom_codigo/preco_com_cupom_centavos/cupom_validado_em` (ver `docs/04`).
- **A CONFIRMAR:** se as APIs expõem validade/preço do cupom e se aplicar cupom cancela a comissão (ver `docs/questoes-em-aberto.md` Q-18).
