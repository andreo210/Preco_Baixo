# Documentação de modelagem — Preço Baixo

Modelagem técnica do sistema em **Mermaid** (renderiza nativo no GitHub/GitLab, sem plugin). Cobre a visão funcional, o modelo de dados, o comportamento das peças plugáveis e os fluxos críticos do **motor** (`coleta 24h → histórico → detecção de queda real`).

> **Fonte da verdade do produto:** os documentos em [`../`](../) (visão, requisitos, arquitetura, modelo de dados, integrações) e o [`CLAUDE.md`](../../CLAUDE.md). **Quando esta modelagem divergir daqueles, eles vencem** — esta pasta é a leitura técnica derivada deles, não uma decisão nova. Decisões arquiteturais estão em [`../decisoes/`](../decisoes/).

## Índice

| Documento | O que mostra | Quando abrir |
|-----------|--------------|--------------|
| [01 — Casos de uso](01-casos-de-uso.md) | Quem interage com o sistema e o que cada um faz (inclui atores não-humanos: agendador, APIs de loja) | Para entender o escopo funcional sem entrar em código |
| [02 — MER (conceitual)](02-mer.md) | As entidades do domínio e como se relacionam, sem tipos/colunas | Para entender o domínio de dados de forma rápida |
| [03 — DER (lógico)](03-der.md) | O schema PostgreSQL com tipos, chaves e restrições | Para implementar/alterar as tabelas e migrações |
| [04 — Diagrama de classes](04-classes.md) | As peças com comportamento: adaptadores de loja/canal, detector, publicador | Para implementar o motor e entender os pontos plugáveis |
| [05 — Diagrama de estados](05-estados.md) | O ciclo de vida de monitoramento do `Produto` | Para entender quando um produto pode/não pode ser publicado |
| [06 — Diagramas de sequência](06-sequencia.md) | 4 fluxos críticos, com o que acontece quando dá errado | Para implementar um fluxo e não errar a ordem das checagens |
| [07 — Dicionário de dados](07-dicionario-de-dados.md) | Campo a campo: tipo, nulo, domínio e a regra que justifica | Referência de consulta rápida no dia a dia |

## Ordem de leitura sugerida

Funcional → domínio → dados → comportamento → fluxo → referência:

**01 casos de uso → 02 MER → 03 DER → 04 classes → 05 estados → 06 sequência → 07 dicionário.**

## Escopo coberto

- O **MVP**: motor (coleta, histórico, detecção de queda real), **canal do Telegram** e **site**.
- Lojas do MVP: **Shopee** e **AliExpress**. Catálogo **curado manualmente** por admin.
- Entidades canônicas: `Loja`, `Produto`, `PrecoHistorico`, `PrecoDiario`, `Publicacao`, `Clique`.

## Fora do escopo (extensão / zona cinzenta — confirmar antes de implementar)

- **Fase 2** — rastreador pessoal por DM: entidades `Usuario` e `Acompanhamento` aparecem no MER/DER/dicionário **marcadas como fase 2**, e o caso de uso correspondente fica de fora. Não implementar no MVP.
- **Nomes de método** nos diagramas de classes e sequência: derivados da arquitetura documentada ([`../03-arquitetura.md`](../03-arquitetura.md)), **ainda não confirmados em código** (o projeto está em planejamento). Estão marcados como propostos.
- **Distinção pausado × removido**: os comandos de admin (`/pausar`, `/remover`) sugerem dois estados, mas o modelo de dados atual só tem `Produto.ativo` (bool). Ver a nota de zona cinzenta em [05 — estados](05-estados.md) e [03 — DER](03-der.md).
- **Itens `A CONFIRMAR`** herdados das integrações (endpoints/campos/rate limits de Shopee e AliExpress, sub_id, validade de cupom via API): consolidados em [`../questoes-em-aberto.md`](../questoes-em-aberto.md). Onde impactam um diagrama, há nota inline.
