# 07 — Site

O site (Next.js, domínio próprio, HTTPS) mostra os produtos com **gráfico de histórico** e **botão "Comprar"** (link de afiliado), além da página de **maiores quedas** e das páginas obrigatórias. Ele dá prova/credibilidade ao canal, captura tráfego (SEO) e permite rastrear cliques. Páginas são **públicas**, mas **nunca** revelam quem acompanha o quê.

## Páginas do MVP

### Página de produto — `/{categoria}/{slug}-{id}`
- Título, imagem e loja.
- **Preço atual** + **preço de referência** + **% de queda** (quando houver).
- **Gráfico do histórico de preços** (lê `PrecoDiario`; ver `docs/04`).
- **Data e hora da última atualização** (obrigatório ao exibir preço).
- Botão **"Comprar"** → redirecionador com `origem=site` → link de afiliado.
- Aviso curto de afiliado + link para a página de aviso completa.
- **Nunca** exibe quantos/quais usuários acompanham o produto.

### Maiores quedas do momento — `/quedas`
- Lista as `Publicacao` recentes ordenadas por % de queda / recência.
- Cada card: produto, de/por, %, loja, "há quanto tempo", botões Comprar e Ver histórico.
- É a "vitrine" pública, espelho do canal.

### Páginas obrigatórias
- **/afiliados** — aviso de que os links são de afiliado (ver `docs/08`).
- **/privacidade** — política de privacidade (LGPD).
- **/termos** — termos de uso.
- Links para as três no rodapé de todas as páginas.

### Home — `/`
- Explica o que é, destaca as maiores quedas e convida a seguir o **canal do Telegram**.

## Integração bot ↔ site (decisão)

Duas rotas possíveis para o alerta/post:
- **(A) Direto para a loja:** menos cliques, maior conversão, mas perde a visita ao site.
- **(B) Para a página do produto no site:** ganha histórico, rastreamento e tráfego, mas adiciona um passo e pode reduzir conversão.

**Decisão (ver `docs/decisoes/0005-destino-do-alerta.md`):** **híbrido** — botão **"Comprar" vai direto para a loja** (converte melhor) e um link **"Ver histórico"** leva à página do site (tráfego + prova + SEO). Melhor dos dois mundos: não sacrifica conversão e ainda alimenta o site.

O canal (e, no futuro, o **canal de promoções** e a fase 2) sempre incluem o "Ver histórico" apontando para o site.

## Privacidade nas páginas públicas

- As páginas são **públicas e indexáveis**, mas expõem **apenas dados do produto e do preço** — nunca usuários.
- No MVP não existe vínculo usuário↔produto. Na **fase 2** (rastreador pessoal) esse vínculo existe no banco, mas **jamais** aparece no site: nada de "X pessoas acompanham", nada que permita inferir indivíduos.
- Cliques são registrados de forma **anonimizada** (ver `docs/04`).

## SEO

- **Indexar:** home, páginas de produto e `/quedas`. **Não indexar:** rotas de admin, redirecionador de clique, endpoints internos (via `robots.txt` + `noindex`).
- **URLs amigáveis:** `/{categoria}/{slug}-{id}` (ex.: `/tenis/tenis-corrida-xyz-123`).
- **Renderização:** **SSR/SSG/ISR** do Next.js para HTML pronto e rápido (bom para indexação e para Core Web Vitals).
- **Performance:** cache/ISR, CDN, imagens otimizadas, HTTPS obrigatório no domínio próprio.
- **`sitemap.xml`** gerado automaticamente (produtos ativos + quedas) e **`robots.txt`**.
- **Metadados:** `<title>`/`<meta description>` por produto, **Open Graph** (para preview bonito no Telegram/redes).
- **Dados estruturados (schema.org):** `Product` + `Offer` com preço e `priceValidUntil`/data de atualização. **A CONFIRMAR:** regras de cada programa de afiliado sobre exibir preço em dados estruturados e a **frequência mínima de atualização** exigida (ver `docs/08`).

## Regras dos programas sobre exibição de preço no site

- Exibir **sempre** a **data/hora da última atualização** junto do preço.
- Atualizar os preços exibidos com a frequência exigida por cada programa. **A CONFIRMAR:** frequência mínima (Shopee, AliExpress).
- Avisos obrigatórios de afiliado visíveis. **A CONFIRMAR:** texto/posição exigidos por cada programa.
- Nunca exibir preço claramente desatualizado como se fosse atual.

## Métricas do site

- **Visitas** por página (home, produto, quedas).
- **Cliques no "Comprar"** com `origem=site` (tabela `Clique`).
- **Vendas/comissões por origem** (canal vs site vs, no futuro, canal de promoções), cruzando os **sub-IDs/tracking IDs** dos programas com os relatórios de conversão. **A CONFIRMAR:** granularidade de sub-id por programa.
- Ver consolidação em `docs/09-roadmap-e-metricas.md`.
- **Analytics:** ferramenta com foco em privacidade (ex.: Plausible/Umami self-hosted) para não coletar dado pessoal além do necessário (LGPD). **A CONFIRMAR:** ferramenta.

## Fases futuras do site

- **Busca** e **categorias** navegáveis.
- **Páginas de comparativo** de produtos otimizadas para SEO (usando o mesmo histórico).
- O **canal de promoções** (fase futura) também aponta para o site.
- Mais nichos além de corrida.
