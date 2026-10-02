# Modelagem — BolsoEmDia

Documentação de modelagem de dados e domínio do sistema de organização financeira pessoal. Os diagramas usam [Mermaid](https://mermaid.js.org/), que o GitHub/GitLab renderiza direto no `.md` — não precisa de ferramenta externa para visualizar, basta abrir o arquivo no navegador do git.

A fonte das regras que justificam cada restrição destes documentos é a skill `regras-negocio-financas` (`.claude/skills/regras-negocio-financas/`) — este diretório é a tradução dessas regras em modelo de dados e classes. Se um documento aqui e a skill divergirem em algum ponto, a skill é a fonte da verdade sobre a regra de negócio; abra uma correção nos dois lugares.

## Documentos

| Documento | O que mostra | Quando abrir |
|---|---|---|
| [`casos-de-uso.md`](casos-de-uso.md) | Diagrama de casos de uso — atores (Usuário / job agendado), funcionalidades e relações `«include»`/`«extend»` | Para uma visão funcional do que o sistema faz, antes de entrar em dados ou classes |
| [`mer.md`](mer.md) | Modelo Entidade-Relacionamento **conceitual** — entidades, relacionamentos e cardinalidades, sem tipos de dado | Para entender o domínio de negócio de ponta a ponta, ou explicar o sistema para alguém não-técnico |
| [`der.md`](der.md) | Diagrama Entidade-Relacionamento **lógico** — atributos com tipo, PK/FK/UK, notas de mapeamento para EF Core | Antes de criar/alterar uma entidade, migration ou configuração do `AppDbContext` |
| [`uml-classes.md`](uml-classes.md) | Diagrama de classes UML no estilo já usado no projeto (`private set`, `static Criar`, agregados, `internal` para entidade interna) | Antes de escrever o código de uma entidade em `BolsoEmDia.Domain/Entidades` — é o esqueleto de onde partir |
| [`diagrama-estados.md`](diagrama-estados.md) | Diagrama de estados de `Fatura` (Aberta/Fechada/Paga) e `Recorrencia` (Ativa/Pausada/Encerrada) — toda transição válida, e por omissão toda transição que não existe | Para tirar dúvida sobre "esse status pode virar aquele outro?" sem precisar caçar isso espalhado nos diagramas de sequência |
| [`diagramas-sequencia.md`](diagramas-sequencia.md) | Diagramas de sequência dos 6 fluxos mais sensíveis (despesa com bloqueio de saldo vs. alerta de orçamento, transferência atômica, compra parcelada com checagem de limite, fechamento de fatura, pagamento de fatura, geração de recorrência) | Antes de implementar o serviço/controller de um desses fluxos — mostra a ordem exata das chamadas entre Controller → Service → Notificador/Entidade → Repositório |
| [`dicionario-dados.md`](dicionario-dados.md) | Dicionário de dados — campo a campo, com domínio de valores e a regra que justifica cada restrição | Para tirar dúvida pontual sobre um campo específico (nulável? unique? qual o range válido?) |

## Ordem sugerida de leitura

1. `casos-de-uso.md` para a visão funcional — o que o sistema faz, para quem.
2. `mer.md` para a visão de domínio — entidades e como se relacionam.
3. `der.md` para o nível de implementação (tipos, chaves).
4. `uml-classes.md` quando for de fato escrever a entidade em C#.
5. `diagrama-estados.md` para o ciclo de vida de `Fatura` e `Recorrencia`.
6. `diagramas-sequencia.md` quando for escrever o serviço/controller de um fluxo específico.
7. `dicionario-dados.md` como referência de consulta enquanto implementa.

## Escopo coberto

Contas (corrente/poupança/carteira/investimento) e transações, transferência entre contas, categorias e orçamento mensal, metas de economia, cartão de crédito com fatura/fechamento/vencimento/limite/parcelamento, e recorrências (transação ou compra de cartão gerada automaticamente).

**Fora do escopo modelado** (mencionado como extensão na skill de regras, não implementar sem confirmar antes): múltiplas moedas, compartilhamento de conta entre mais de um usuário, integração com extrato bancário (open finance/OFX), teto configurável de cheque especial, taxa de transferência, estorno parcial de compra parcelada já paga parcialmente.
