# 02 — Requisitos

Requisitos funcionais (RF) e não funcionais (RNF) do MVP, numerados e rastreáveis. O MVP cobre o **motor** (coleta, histórico, detecção de queda real), o **canal do Telegram** e o **site**. Itens de fases futuras aparecem como "Fora do MVP".

## Requisitos funcionais — Catálogo e coleta

- **RF-01** — O admin adiciona um produto ao catálogo enviando o link ao bot (modo admin). O sistema identifica **loja** e **produto** pelo link.
- **RF-02** — O admin lista, pausa, retoma e remove produtos do catálogo.
- **RF-03** — O sistema coleta periodicamente o **preço** e a **disponibilidade** de cada produto ativo, via **API oficial** da loja.
- **RF-04** — A frequência de verificação é **adaptativa**: produtos com mais interesse (mais cliques) ou preço mais volátil são checados com mais frequência; tudo dentro dos limites de rate de cada API (ver RNF-05).
- **RF-05** — O sistema guarda o **histórico de preços** gravando um novo registro **apenas quando o preço muda**, mais um registro diário de "continua vivo" (ver retenção em `docs/04`).
- **RF-06** — O preço é sempre associado à **variação correta** (SKU/tamanho/cor). O sistema nunca compara variações diferentes.

## Requisitos funcionais — Detecção de queda real

- **RF-07** — O sistema calcula um **preço de referência** por produto/variação a partir do histórico (ex.: mediana/percentil dos últimos 30 dias) — ver definição exata em `docs/05`.
- **RF-08** — Uma **queda real** só é reconhecida se: (a) o produto está **disponível**; (b) o preço atual é menor que o de referência por uma **margem em camadas** (Boa oferta 10–20% / Queda forte ≥20%) e ≥ R$ mínimo — ver `docs/05`; (c) é a **mesma variação**; (d) respeita o **cooldown** anti-repetição.
- **RF-09** — O sistema ignora quedas causadas por **item esgotado**, **variação diferente**, **cupom** não comparável e **frete** (ver critérios em `docs/05`).
- **RF-10** — Ao reconhecer uma queda real, o sistema gera o **link de afiliado** da loja e cria uma **Publicacao**.

## Requisitos funcionais — Canal do Telegram

- **RF-11** — O sistema publica a queda no **canal público** com: título do produto, **preço atual**, **preço de referência**, **percentual de queda** (sobre o preço base), **nível** (Boa oferta / Queda forte), **data/hora da última atualização**, imagem e botão **"Comprar"** (link de afiliado direto para a loja).
- **RF-12** — O post inclui um link **"Ver histórico"** para a página do produto no site.
- **RF-13** — Todo post (ou a descrição fixa do canal) informa que os links são de **afiliado**.
- **RF-14** — O sistema respeita os **limites do Telegram** (ver RNF-05) e não republica o mesmo produto dentro do cooldown.
- **RF-15** — Comandos de **admin** pelo bot: adicionar/listar/pausar/retomar/remover produto, status do sistema e publicação manual (ver `docs/06`).

## Requisitos funcionais — Site

- **RF-16** — O site tem uma **página por produto** com: preço atual, **gráfico do histórico**, data/hora da última atualização e botão **"Comprar"** (link de afiliado).
- **RF-17** — O site tem uma página com as **maiores quedas do momento**.
- **RF-18** — O site tem as páginas obrigatórias: **aviso de afiliado**, **política de privacidade** e **termos de uso**.
- **RF-19** — As páginas de produto são **públicas** e **nunca** revelam quais usuários acompanham quais produtos.
- **RF-20** — As páginas públicas (produto, maiores quedas) são **indexáveis** pelo Google, com URLs amigáveis; páginas administrativas **não** são indexáveis (ver `docs/07`).

## Requisitos funcionais — Rastreamento e métricas

- **RF-21** — Cliques no botão "Comprar" são rastreados por **origem** (canal, site) usando **sub-ID / tracking ID** do programa, quando o programa permitir.
- **RF-22** — O redirecionamento de clique pode passar pelo **domínio próprio** apenas se as regras do programa permitirem; caso contrário, usa-se o link direto do programa (ver `docs/08`).
- **RF-23** — O sistema registra métricas do MVP: inscritos no canal, produtos no catálogo, posts publicados, cliques por origem e (via relatórios dos programas) vendas/comissões e visitas ao site.

## Requisitos funcionais — LGPD / conformidade

- **RF-24** — O sistema coleta o **mínimo** de dados pessoais. No MVP (canal broadcast) praticamente não há dado pessoal; cliques são registrados de forma **anonimizada**.
- **RF-25** — O sistema exibe **política de privacidade**, **termos de uso** e **aviso de afiliado** no site.
- **RF-26** — (Prepara fase 2) Haverá comando para o usuário **apagar os próprios dados** quando existir cadastro de usuário.

## Requisitos não funcionais

- **RNF-01 — Disponibilidade:** a coleta roda **24h**. Falha de uma loja não derruba as demais (adaptadores isolados).
- **RNF-02 — Confiabilidade da oferta:** taxa de **quedas falsas** publicadas próxima de zero é prioridade sobre volume de posts.
- **RNF-03 — Desempenho do site:** páginas públicas rápidas (SSR/SSG + cache), HTTPS no domínio próprio, boa nota de performance (ver metas em `docs/07`).
- **RNF-04 — Escalabilidade:** arquitetura suporta adicionar **lojas** e **canais** via adaptadores sem reescrever o núcleo; banco cresce de forma previsível (delta + downsampling, ver `docs/04`).
- **RNF-05 — Respeito a limites de API:** nunca exceder os limites do Telegram (~1 msg/s por chat, 20 msg/min por grupo, ~30 msg/s global) nem os limites de rate das lojas (**A CONFIRMAR** por loja). Orçamento de chamadas por loja.
- **RNF-06 — Conformidade:** cumprir regras de cada programa de afiliado (exibição/armazenamento de preço, redirecionamento, aviso de afiliado), **LGPD** e termos do Telegram. Preferir API oficial a scraping.
- **RNF-07 — Observabilidade:** logs estruturados, healthchecks e monitoramento com alerta de falha (ver `docs/03`).
- **RNF-08 — Backup:** backup diário do banco (histórico é ativo estratégico), com retenção e restauração testada.
- **RNF-09 — Segurança:** segredos (tokens, chaves de API) fora do código, em variáveis de ambiente; HTTPS obrigatório; acesso admin restrito.
- **RNF-10 — Custo:** operar dentro de um VPS ~4 vCPU / 8 GB no MVP; escolhas que evitem custo por mensagem (por isso Telegram no MVP, não WhatsApp).
- **RNF-11 — Autoatendimento:** mensagens e páginas devem resolver dúvidas sozinhas; nenhum canal de atendimento humano.
- **RNF-12 — Manutenibilidade:** um adaptador por loja e por canal; nomes de entidades canônicos (ver `CLAUDE.md` e `docs/04`).

## Fora do MVP

- **FDM-01** — Rastreador pessoal por DM (usuário vigia produto próprio) — fase 2. Inclui **limite de produtos por usuário** (proposta: **50**) e comando `/apagardados`.
- **FDM-02** — **WhatsApp** como canal — fase futura, condicionado à economia (custo de template Meta vs comissão).
- **FDM-03** — **Feeds automáticos** de catálogo (Shopee `productOfferV2`, hot products AliExpress) — fase futura.
- **FDM-04** — **Amazon** (Creators API, exige vendas qualificadas) e **Mercado Livre** (sem API oficial de afiliados) — fases futuras.
- **FDM-05** — Site: **busca**, **categorias** e **páginas de comparativo** para SEO — fase futura.
- **FDM-06** — Mais nichos além de corrida — fase futura.
