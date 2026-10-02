# 01 — Visão Geral

O Preço Baixo é um sistema que coleta preços 24h, guarda o histórico e publica **apenas quedas de preço reais** — no nicho de **corrida** — num **canal do Telegram** e num **site**. É grátis para o usuário e monetizado por **links de afiliado**. Tudo é autoatendimento: **suporte quase zero**.

## Problema

Quem compra artigos de corrida (tênis, relógios/GPS, fones, roupa técnica, acessórios) quer pagar barato, mas:
- Não sabe se o "desconto" é real ou preço inflado com falso corte.
- Não tem tempo de acompanhar o preço de vários produtos todo dia.
- Os canais de oferta existentes em geral **só repostam** links de afiliado, sem verificar se o preço caiu de verdade.

## Proposta de valor

> **"Só ofertas de verdade: quedas de preço conferidas no nosso próprio histórico."**

O diferencial não é "achar barato" — é **provar** a queda contra o histórico real dos últimos dias/semanas (ex.: "caiu 28% em relação ao preço normal dos últimos 30 dias"). Isso gera confiança, que é o ativo que os canais genéricos não têm.

## Público

- **Primário (MVP):** corredores e entusiastas de corrida que compram em marketplaces (Shopee, AliExpress) e participam de grupos/comunidades de corrida.
- Sensíveis a preço, acostumados a Telegram, dispostos a esperar o preço cair.

**Premissa:** o público de corrida tem comunidades ativas e de fácil acesso para o lançamento (grupos de corrida, assessorias, redes sociais de corredores).

## Modelo de receita

- **Comissão de afiliado:** a loja paga um percentual quando alguém compra pelo nosso link. O usuário **nunca paga**.
- No MVP, receita vem dos cliques no **canal** e no **site** que viram venda na **Shopee** e no **AliExpress**.
- Rastreamento por **sub-ID / tracking ID** de cada programa para saber a origem (canal vs site) — ver `docs/07-site.md` e `docs/05-integracoes.md`.

## Os dois produtos rodam no mesmo motor

```
coleta 24h  →  histórico  →  detecção de QUEDA REAL  →  [ saída ]
```

- **Saída do MVP:** canal de promoções (broadcast, um-para-muitos) + páginas do site.
- **Saída da fase 2:** rastreador pessoal (um-para-um, o usuário escolhe o produto e é avisado).

Trocar/adicionar saída **não** mexe no motor. O mesmo vale para lojas e canais (adaptadores plugáveis).

## Concorrência e diferenciação

| Concorrente | O que faz | Onde temos brecha |
|-------------|-----------|-------------------|
| Buscapé, Zoom | Comparação + histórico + alerta por e-mail/site (pull) | Foco em grandes varejistas; nicho de corrida em Shopee/AliExpress é mal coberto; push no Telegram engaja mais. |
| Promobit, Pelando, canais de oferta no Telegram | Comunidade/canais repostando ofertas | A maioria **não verifica** se é queda real; nós verificamos contra o histórico. Identidade de nicho ("corrida") vs canal genérico. |
| Mercado Livre, Netshoes | Loja/marketplace | Não são neutros nem agregam ofertas entre lojas. |

**Aposta central:** nicho (corrida) + push no Telegram + **queda verificada** + fricção zero. Não é "mais um canal genérico de promoção".

**Risco #1 (a validar):** existe gente disposta a seguir e comprar pelo nosso canal em vez de usar Buscapé/Promobit? A validação ocorre na Semana 4 e a decisão aos ~60 dias — ver `docs/09-roadmap-e-metricas.md`.

## Princípios

1. **Autoatendimento, suporte quase zero.** Mensagens claras, erros amigáveis, nenhum canal de atendimento humano.
2. **Confiança antes de volume.** Melhor postar pouca oferta 100% real do que muita duvidosa.
3. **Histórico é ativo estratégico.** Guardamos e cuidamos dos dados de preço; eles alimentam canal, site e fases futuras.
4. **APIs oficiais antes de scraping.** Nunca coletar onde os termos proíbem.
5. **Adaptadores plugáveis.** Nova loja ou novo canal entra sem reescrever o núcleo.
6. **Privacidade por padrão.** Coletar o mínimo; nunca expor quem acompanha o quê.
7. **Começar estreito.** Um nicho bem servido vale mais que "qualquer produto" mal servido.

## Escopo do MVP x fases futuras (resumo)

- **MVP:** motor + canal de corrida + site (produto, maiores quedas, páginas obrigatórias). Lojas: Shopee + AliExpress. Catálogo curado manualmente.
- **Fase 2:** rastreador pessoal por DM; WhatsApp (condicionado à economia); feeds automáticos de catálogo.
- **Fase 3:** mais lojas (Amazon/Creators, Mercado Livre), mais nichos, busca/categorias e páginas de comparativo para SEO.

Detalhes e critérios em `docs/09-roadmap-e-metricas.md`.
