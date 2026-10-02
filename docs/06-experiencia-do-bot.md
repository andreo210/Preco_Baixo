# 06 — Experiência do Bot e do Canal

Como o produto se comunica: o **formato dos posts no canal**, os **comandos de admin** do bot e os **textos das mensagens** (sucesso, erro, FAQ). No MVP o "bot" é sobretudo um **publicador** + painel de admin. Comandos e fluxos de usuário por DM estão marcados como **fase 2**. Tudo em PT-BR, tom direto, **autoatendimento** (nenhum atendimento humano).

## Canal público (a cara do produto)

- **Nome/identidade:** canal de ofertas de **corrida** ("só quedas de verdade, conferidas no histórico").
- **Descrição fixa do canal** (inclui o aviso de afiliado obrigatório):

> 🏃 Ofertas reais de corrida (tênis, relógios, fones, roupa e acessórios).
> Só publicamos quando o preço **cai de verdade** — conferido no nosso histórico.
> 🔗 Os links são de **afiliado**: você paga o mesmo preço e a gente ganha uma comissão da loja. Isso mantém o canal grátis.
> 📊 Histórico de cada produto no site: precobaixo.com.br *(A CONFIRMAR: domínio)*

### Formato do post de oferta

```
🔥 QUEDA FORTE · -28%

Tênis de Corrida XYZ (tam. 40/41/42)
De R$ 349,90  →  por R$ 249,90
📉 Abaixo do preço normal dos últimos 30 dias (referência: R$ 349,90)
🎟️ Bônus: + cupom CORRIDA10 (validado agora · pode expirar/ter limite)
🏪 Shopee · atualizado há 12 min

[ 🛒 Comprar ]   [ 📊 Ver histórico ]
```

Regras do post (ligam aos requisitos):
- Mostra **preço atual**, **preço de referência**, **% de queda** e **data/hora da última atualização** (RF-11).
- **Comprar** → link de afiliado **direto para a loja** (maior conversão), via redirecionador com `origem=canal` (RF-11, RF-21).
- **Ver histórico** → página do produto no site (RF-12).
- Aviso de afiliado garantido pela descrição fixa do canal (RF-13).
- Nunca republica dentro do cooldown (RF-14).
- **Nível do selo (ADR 0011):** **🔥 QUEDA FORTE** para queda ≥20%; **✅ Boa oferta** para 10–20%. O % da manchete é sempre sobre o **preço base**.
- **Cupom (ADR 0010):** só aparece como **bônus** se **validado via API no momento da publicação**, sempre com a ressalva "pode expirar/ter limite"; **nunca** entra no % (que é base vs base). Ver `docs/05`.

## Comandos de admin (MVP)

Acesso restrito por **ID de usuário do Telegram** numa allowlist. Usados em DM com o bot.

| Comando | O que faz |
|---------|-----------|
| `/add <link>` | Adiciona um produto ao catálogo (identifica loja e produto). |
| `/listar [categoria]` | Lista produtos monitorados (com preço atual e status). |
| `/pausar <id>` | Pausa o monitoramento de um produto (não apaga histórico). |
| `/retomar <id>` | Retoma um produto pausado. |
| `/remover <id>` | Desativa o produto (mantém histórico). |
| `/status` | Saúde do sistema: nº de produtos ativos, última coleta por loja, tamanho da fila, erros recentes. |
| `/publicar <id>` | Força a publicação de uma oferta (bypass manual, respeitando aviso e formato). |
| `/ajuda` | Lista os comandos de admin. |

### Textos de admin (exemplos)

- Sucesso ao adicionar: `✅ Adicionado: "{título}" ({loja}). Preço atual: R$ {preço}. Vou acompanhar.`
- Link não reconhecido: `❓ Não consegui identificar a loja/produto nesse link. No MVP eu entendo links da Shopee e do AliExpress. Confere se o link está completo.`
- Produto já existe: `ℹ️ Esse produto já está no monitoramento (id {id}).`
- Falha na loja: `⚠️ A loja não respondeu agora. Registrei e vou tentar de novo automaticamente.`

## Fluxos de usuário por DM — FASE 2 (documentado, não implementado)

Rastreador pessoal: o usuário manda um link e é avisado quando **aquele** produto cair.

| Comando (fase 2) | O que faz |
|------------------|-----------|
| `/start` | Boas-vindas + explica o que o bot faz + aviso de afiliado + link da política de privacidade. |
| `/monitorar <link>` | Passa a vigiar o produto para aquele usuário (limite **50** produtos ativos). |
| `/alvo <id> <preço>` | Define um preço-alvo para alerta. |
| `/meus` | Lista os produtos que o usuário acompanha. |
| `/remover <id>` | Para de acompanhar um produto. |
| `/apagardados` | Apaga todos os dados do usuário (LGPD). Confirmação em duas etapas. |
| `/ajuda` | FAQ + comandos. |

Texto ao atingir o limite (fase 2):
> Você chegou ao limite de **50 produtos** monitorados. Remova algum com `/remover` para adicionar outro. (Esse limite existe para o serviço continuar rápido e grátis.)

## Mensagens de erro (princípios)

- Sempre dizer **o que aconteceu** e **o que fazer** — nunca um erro técnico cru.
- Nunca pedir para "entrar em contato com o suporte" (não existe suporte humano). Direcionar para a **FAQ** e comandos.
- Tom calmo e curto.

## FAQ (para a descrição do canal / página do site / `/ajuda`)

**O link é de afiliado? Eu pago mais caro?**
Não. O preço é o mesmo. A loja nos paga uma comissão quando você compra pelo link — é isso que mantém tudo grátis.

**Como vocês sabem que a queda é real?**
Guardamos o histórico de preço de cada produto. Só postamos quando o preço cai de verdade em relação ao preço normal recente (não vale "desconto" inflado).

**Com que frequência vocês verificam os preços?**
O tempo todo, de forma automática. Produtos mais procurados são checados com mais frequência.

**Vocês cobrem quais lojas?**
No momento, Shopee e AliExpress. Outras lojas devem entrar depois.

**Achei um preço diferente do post.**
O preço muda o tempo todo na loja. Por isso mostramos a **hora da última atualização** e o link direto — confira sempre na loja antes de comprar.

**Como vejo o histórico de um produto?**
Clique em **"Ver histórico"** no post para abrir a página do produto no site, com o gráfico.

**Posso pedir para acompanhar um produto específico?** *(fase 2)*
Em breve. Estamos preparando um recurso em que você manda o link e a gente te avisa quando aquele produto cair.
