# Registros de Decisão (ADRs)

Registros curtos das decisões importantes do projeto. Cada arquivo segue o formato: **Contexto → Decisão → Alternativas → Consequências**.

| # | Decisão |
|---|---------|
| [0001](0001-stack-tecnologica.md) | Stack: Python (aiogram/Celery) + Next.js + PostgreSQL + Redis |
| [0002](0002-telegram-como-canal-mvp.md) | Telegram como canal do MVP; WhatsApp fora |
| [0003](0003-adaptadores-plugaveis.md) | Loja e canal de notificação como adaptadores plugáveis |
| [0004](0004-lojas-do-mvp.md) | Shopee + AliExpress no MVP; Amazon e Mercado Livre depois |
| [0005](0005-destino-do-alerta.md) | "Comprar" direto na loja + "Ver histórico" no site |
| [0006](0006-guardar-historico-de-precos.md) | Guardar histórico com delta + downsampling |
| [0007](0007-mvp-canal-de-promocoes-primeiro.md) | MVP = canal de promoções; rastreador pessoal na fase 2 |
| [0008](0008-nicho-corrida.md) | Nicho do MVP: corrida (não "qualquer produto", nem "só tênis") |
| [0009](0009-catalogo-curado-manual.md) | Catálogo curado manualmente no MVP; feeds automáticos depois |
| [0010](0010-tratamento-de-cupom.md) | Cupom: manchete só no preço base; bônus validado antes de postar |
| [0011](0011-margem-em-camadas.md) | Margem em camadas; volume via catálogo, não via régua mais baixa |
