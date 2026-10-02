# CLAUDE.md — Contexto do projeto Preço Baixo

Contexto essencial para as próximas sessões. Leia isto antes de codar. Documentação completa em `docs/`.

## O que é

Rastreador de preços com **canal de ofertas no Telegram** + **site**, monetizado por **links de afiliado**. Coleta preços 24h, guarda histórico e publica **só quedas reais** (verificadas contra o próprio histórico). Nicho do MVP: **corrida**. Usuário usa de graça; receita = comissão das lojas. Princípio-mãe: **autoatendimento, suporte quase zero**.

## Escopo do MVP (o que está EM jogo agora)

Um **motor** único alimenta tudo: `coleta 24h → histórico → detecção de queda real`. A saída do MVP é:
- **Canal público do Telegram** que posta as maiores quedas reais de produtos de corrida.
- **Site** com página por produto (gráfico de histórico), página de maiores quedas e páginas obrigatórias (aviso de afiliado, privacidade, termos).

Catálogo é **curado manualmente** (admin adiciona produtos por link). Lojas do MVP: **Shopee** e **AliExpress**.

## Fora do MVP (fases futuras — não implementar agora)

- **Rastreador pessoal por DM** (usuário manda link, bot vigia só pra ele e alerta) — **fase 2**.
- **WhatsApp** — fase futura, só se a economia fechar (custo de template Meta vs comissão).
- **Feeds automáticos** de catálogo (Shopee `productOfferV2`, hot products AliExpress) — fase futura.
- **Amazon** (Creators API exige vendas qualificadas) e **Mercado Livre** (sem API oficial de afiliados) — fases futuras.
- Canal aponta para o site; busca, categorias e páginas de comparativo SEO — fases futuras.

## Stack

- **Python**: bot (**aiogram**), API interna/admin (**FastAPI**), tarefas/agendamento (**Celery + Celery Beat + Redis**), coletor e adaptadores de loja.
- **Next.js** (TypeScript): site (SSR/SSG para SEO), consome a **API FastAPI**, domínio próprio + HTTPS.
- **PostgreSQL**: dados e histórico de preços.
- **Redis**: broker do Celery + cache.
- **Docker Compose** em VPS Linux (Ubuntu LTS, ~4 vCPU / 8 GB).
- Reverse proxy com HTTPS (Caddy ou nginx + Let's Encrypt). **A CONFIRMAR:** escolha do proxy.

## Estrutura de pastas planejada (quando o código existir)

```
/                 README.md, CLAUDE.md, docker-compose.yml, .env.example
/docs             documentação (fonte da verdade do produto)
/backend          Python: bot, worker, coletor, adaptadores de loja, modelos
  /adapters       um módulo por loja (shopee, aliexpress, ...) + interface comum
  /channels       um módulo por canal de notificação (telegram, ...) — plugável
  /core           motor: coleta, detecção de queda real, histórico
  /models         entidades / ORM
/site             Next.js (páginas públicas + SEO)
/infra            Dockerfiles, compose, scripts de backup e deploy
```

## Entidades principais (nomes canônicos — use SEMPRE estes)

`Loja`, `Produto`, `PrecoHistorico`, `PrecoDiario` (downsample), `Publicacao` (post no canal), `Clique`. Fase 2: `Usuario`, `Acompanhamento`. Detalhes em `docs/04-modelo-de-dados.md`.

## Regras — SEMPRE

- **SEMPRE** comparar preço da **mesma variação** (mesmo SKU/tamanho/cor). Nunca comparar variações diferentes.
- **SEMPRE** guardar preço em **inteiro (centavos)**.
- **SEMPRE** gravar no histórico **só quando o preço muda** (+ 1 registro diário de "vivo"). Ver retenção em `docs/04`.
- **SEMPRE** exibir data/hora da última atualização ao mostrar preço (canal e site).
- **SEMPRE** informar que o link é de afiliado (canal e site).
- **SEMPRE** preferir **API oficial**; nunca fazer scraping onde os termos proíbem coleta.
- **SEMPRE** tratar **canal de notificação** e **loja** como **adaptadores plugáveis**.

## Regras — NUNCA

- **NUNCA** postar queda sem confirmar que é real (ver definição em `docs/05`): respeitar limiar %/R$, disponibilidade, mesma variação, cooldown anti-repetição.
- **NUNCA** contar cupom/frete de forma inconsistente na comparação de preço.
- **NUNCA** expor no site/canal **quais usuários acompanham quais produtos** (mesmo na fase 2).
- **NUNCA** guardar dado pessoal além do mínimo (LGPD). No MVP quase não há dado pessoal.
- **NUNCA** estourar os limites das APIs (Telegram e lojas) — usar frequência adaptativa e orçamento de chamadas.
- **NUNCA** criar canal de atendimento humano; tudo é autoatendimento.

## Decisão-chave: destino do clique

Post do canal traz **botão "Comprar"** que vai **direto para a loja** (link de afiliado, mais conversão) e um link **"Ver histórico"** para a página do produto no site. Ver `docs/decisoes/0005-destino-do-alerta.md`.

## Marcadores

Use `Premissa:` para suposições e `A CONFIRMAR:` para o que não foi verificado em fonte oficial (consolidado em `docs/questoes-em-aberto.md`).
