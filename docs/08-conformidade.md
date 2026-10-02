# 08 — Conformidade

Regras dos programas de afiliados (exibição/armazenamento de preço, redirecionamento, aviso de afiliado), **LGPD** (dado mínimo, exclusão de dados, política de privacidade), limites do Telegram e a preferência por **API oficial sobre scraping**. Fecha com esboços de **termos de uso** e **política de privacidade**. Muitos itens dependem dos termos de cada programa e estão como **A CONFIRMAR** (Semana 1).

## Princípios de conformidade

1. **API oficial antes de scraping.** Nunca coletar onde os termos proíbem. Loja sem caminho oficial fica fora (ver Mercado Livre em `docs/05`).
2. **Dado mínimo (LGPD).** Coletar só o necessário. No MVP (canal broadcast) quase não há dado pessoal.
3. **Transparência.** Deixar claro que os links são de afiliado, em todo lugar relevante.
4. **Respeitar limites.** Telegram e APIs das lojas (ver `docs/05`).

## Regras dos programas de afiliados

Cada programa tem regras próprias; abaixo o que precisa ser cumprido e o que falta confirmar.

### Aviso de afiliado (divulgação)
- Informar que o link é de afiliado é **obrigatório** na maioria dos programas e é boa prática/consumerista.
- **Onde cumprimos:** descrição fixa do **canal** (`docs/06`), rodapé e página **/afiliados** do **site** (`docs/07`).
- **A CONFIRMAR:** texto e posicionamento exatos exigidos por Shopee e AliExpress.

### Exibição e armazenamento de preço
- Exibir **data/hora da última atualização** junto do preço (canal e site).
- Atualizar preços com a **frequência mínima** exigida por cada programa. **A CONFIRMAR** (Shopee, AliExpress).
- **Armazenar** histórico de preço: **A CONFIRMAR** se algum programa restringe guardar/exibir histórico. (Guardamos para uso próprio; exibição segue as regras.)
- Não exibir preço desatualizado como atual.

### Redirecionamento / cloaking
- Rastreamento de clique pelo **domínio próprio** (redirecionador 302) só se o programa **permitir**. Alguns programas restringem "cloaking"/redirecionamentos.
- **A CONFIRMAR** por programa (Shopee, AliExpress). Se proibido, usar o **link direto do programa** sem redirecionador próprio (e rastrear via sub-id do programa).

### Sub-ID / tracking de origem
- Usar sub-id/tracking id para separar **canal vs site** (e futuras origens). Depende de suporte do programa. **A CONFIRMAR** (ver `docs/05`).

### Marcas restritas
- Alguns programas **excluem certas marcas** (ex.: calçados de marca famosa) ou categorias. Relevante para o nicho de corrida.
- **A CONFIRMAR:** lista de marcas/categorias restritas em Shopee e AliExpress.

## LGPD

### Dados tratados
- **MVP (canal broadcast):** praticamente **nenhum dado pessoal**. O Telegram gerencia os inscritos do canal; não armazenamos quem são. Cliques são **anonimizados** (sem identificar a pessoa; `ip_hash`/`ua_hash` só se necessários para antiabuso — **A CONFIRMAR** necessidade e base legal).
- **Fase 2 (rastreador pessoal):** armazenamos o **telegram_user_id** e os produtos que a pessoa acompanha. Aí a LGPD pesa de verdade.

### Bases legais e direitos
- **Base legal:** para o MVP, legítimo interesse / execução do serviço; para a fase 2, execução do serviço solicitado pelo usuário. **A CONFIRMAR** com revisão jurídica.
- **Minimização:** só coletamos o essencial para o serviço funcionar.
- **Exclusão:** comando **`/apagardados`** (fase 2) apaga todos os dados do usuário; soft-delete imediato + expurgo definitivo em janela curta. No MVP não há dado pessoal de usuário a apagar.
- **Transparência:** **política de privacidade** pública no site explica o que é coletado, por quê e como pedir exclusão.
- **Retenção:** dados pessoais pelo tempo mínimo; histórico de **preço** (não pessoal) é retido como ativo (ver `docs/04`).

### Encarregado / contato
- Como não há atendimento humano, disponibilizar um **canal de solicitação LGPD** (e-mail/formulário) apenas para direitos do titular — o mínimo exigido por lei. **A CONFIRMAR:** forma (e-mail dedicado?).

## Limites do Telegram

- Respeitar ~1 msg/s por chat, 20 msg/min por grupo, ~30 msg/s global (fonte oficial; ver `docs/05`). No MVP o volume é baixo. A **fase 2** (push 1-a-1) exige fila com controle de taxa.

## Termos de uso (esboço — a revisar juridicamente)

Pontos a cobrir em **/termos**:
- O serviço é **informativo**; preços podem estar **desatualizados** — confira sempre na loja antes de comprar.
- Não vendemos produtos; apenas indicamos ofertas com **links de afiliado**.
- Sem garantia de disponibilidade do preço mostrado.
- Uso do canal/site implica aceitação dos termos.
- Podemos alterar/descontinuar o serviço.
- Limitação de responsabilidade por decisões de compra.
- **A CONFIRMAR:** revisão jurídica.

## Política de privacidade (esboço — a revisar juridicamente)

Pontos a cobrir em **/privacidade**:
- Quais dados coletamos (MVP: navegação/cliques anonimizados; fase 2: id do Telegram e produtos acompanhados).
- Para quê (operar o serviço e medir desempenho de ofertas).
- Com quem compartilhamos (programas de afiliados recebem o clique/sub-id; ferramenta de analytics com foco em privacidade).
- Cookies/analytics e como recusar.
- Direitos do titular e como exercê-los (inclui `/apagardados` na fase 2).
- Retenção e segurança.
- **A CONFIRMAR:** revisão jurídica e ferramenta de analytics.

## Checklist de conformidade para o lançamento (Semana 4)

- [ ] Descrição do canal com aviso de afiliado.
- [ ] Páginas /afiliados, /privacidade e /termos publicadas.
- [ ] Data/hora de atualização visível em todo preço (canal e site).
- [ ] Redirecionador conforme regra do programa (ou link direto).
- [ ] Sub-id de origem configurado (se suportado).
- [ ] Nenhum dado pessoal além do mínimo; cliques anonimizados.
- [ ] Confirmados os "A CONFIRMAR" de Shopee e AliExpress (ver `docs/questoes-em-aberto.md`).
