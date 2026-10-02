# 0002 — Telegram como canal do MVP (WhatsApp fora)

**Status:** aceita · **Data:** 2026-09-30

## Contexto
No Brasil o WhatsApp tem alcance muito maior que o Telegram. Mas o produto vive de **push** (avisar quando o preço cai), e o modelo é grátis para o usuário, monetizado só por comissão.

## Decisão
Usar **Telegram** no MVP (canal público + bot). **WhatsApp fica fora** do MVP.

## Alternativas
- **WhatsApp não oficial** (automação do WhatsApp Web): viola os termos da Meta → **risco de banir o número**. Descartado.
- **WhatsApp Business Platform / Cloud API** (oficial): legítimo, porém a **janela de 24h** obriga usar **templates pagos** para mensagens proativas fora dela — exatamente o caso do alerta de queda. Cada alerta custaria por mensagem à Meta, inclusive os que não convertem, ameaçando a economia (comissão baixa). **A CONFIRMAR:** preços atuais de template/conversa.

## Consequências
- Telegram Bot API é **gratuita**, sem custo por mensagem e sem janela de 24h para broadcast → casa com o modelo de receita.
- Menor alcance que WhatsApp; parte do risco de tração (ver `docs/01`).
- WhatsApp volta como **fase futura condicionada à economia** (comissão por alerta > custo do template). O canal é **plugável** (ver ADR 0003), então adicionar não reescreve o núcleo.
