# Dicionário de dados

Mermaid não tem um tipo de diagrama para dicionário de dados (é tabela, não grafo), então este documento é markdown puro — renderiza igual no GitHub/GitLab, só não é um bloco ` ```mermaid `. Ele detalha campo a campo o que o [DER](der.md) mostra de forma visual. Onde uma regra de negócio explica o "por quê" de uma restrição, ela é referenciada entre parênteses — o texto completo da regra está na skill `regras-negocio-financas`.

## Convenções válidas para todas as tabelas abaixo

- **Chave primária**: `IdEntidade`, `int`, autoincremento (identity no banco).
- **Escopo por usuário**: toda entidade abaixo, exceto `ApplicationUser`, tem uma coluna `IdUsuario` (`string`, FK para `AspNetUsers.Id`, obrigatória). Ela não é repetida tabela por tabela para não poluir — está implícita em "toda entidade de negócio pertence a um usuário" (`IPertenceAoUsuario`, filtro global de query).
- **Auditoria**: toda entidade de negócio também tem, via `IAuditoria`: `DataCriacao` (`datetime`, obrigatória), `IdUsuarioCriacao` (`string`, nullable — `"SYSTEM"` para jobs/seed), `DataModificacao` (`datetime`, nullable), `IdUsuarioModificacao` (`string`, nullable). Também omitidas linha a linha pelo mesmo motivo.
- Datas são gravadas em UTC (`DateTime.UtcNow`), nunca hora local.
- Valores monetários (`decimal`) são sempre armazenados **positivos**; o sinal/efeito no saldo vem do campo `Tipo` da transação, nunca do sinal do número.

---

## Conta

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdConta | int | não | PK | autoincremento | Identificador da conta |
| Nome | string(100) | não | | livre | Nome dado pelo usuário (ex.: "Nubank", "Carteira") |
| Tipo | int (enum) | não | | `Corrente=0`, `Poupanca=1`, `Carteira=2`, `Investimento=3` | Define a regra de saldo negativo (ver abaixo) |
| SaldoInicial | decimal(18,2) | não | | ≥ qualquer valor (pode ser negativo só se `Tipo=Corrente`) | Saldo no momento em que a conta foi cadastrada no sistema — saldo atual = `SaldoInicial + soma das transações efetivadas` |
| Ativa | bool | não | | `true`/`false` | Conta inativa não aparece para novo lançamento, mas mantém histórico |

**Regra de saldo negativo**: apenas `Tipo=Corrente` pode ter saldo resultante negativo (cheque especial). `Poupanca`, `Carteira` e `Investimento` bloqueiam qualquer lançamento que resultaria em saldo `< 0`. Ver `contas-e-transacoes.md` na skill de regras de negócio.

---

## Categoria

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdCategoria | int | não | PK | autoincremento | |
| Nome | string(100) | não | | livre | |
| Tipo | int (enum) | não | | `Receita=0`, `Despesa=1` | Uma transação só pode usar categoria do mesmo `Tipo` que ela |
| IdCategoriaPai | int | sim | FK → Categoria.IdCategoria | | Hierarquia de **um nível só** — uma subcategoria não pode ter `IdCategoriaPai` apontando para outra subcategoria (validar na criação) |
| Ativa | bool | não | | `true`/`false` | Excluir categoria com transações lançadas é bloqueado; usar `Ativa=false` para "arquivar" |

---

## Transacao

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdTransacao | int | não | PK | autoincremento | |
| IdConta | int | não | FK → Conta.IdConta | | Conta afetada pelo lançamento |
| IdCategoria | int | sim | FK → Categoria.IdCategoria | | **Obrigatório** quando `Tipo` é `Receita`/`Despesa`; **sempre nulo** quando `Tipo` é `TransferenciaSaida`/`TransferenciaEntrada` |
| IdTransferencia | int | sim | FK → Transferencia.IdTransferencia | | Preenchido só nas duas pernas geradas por uma transferência |
| IdRecorrencia | int | sim | FK → Recorrencia.IdRecorrencia | | Preenchido quando a transação foi gerada automaticamente por um modelo de recorrência |
| Tipo | int (enum) | não | | `Receita=0`, `Despesa=1`, `TransferenciaSaida=2`, `TransferenciaEntrada=3` | Define o efeito no saldo: Receita/TransferenciaEntrada somam, Despesa/TransferenciaSaida subtraem |
| Data | date | não | | passada, hoje ou futura | Se `Data > hoje`, a transação é "agendada" e **não** entra no cálculo de saldo atual (só no saldo projetado) |
| Valor | decimal(18,2) | não | | `> 0` | Sempre positivo — o sinal vem de `Tipo` |
| Descricao | string(255) | sim | | livre | |

**Nunca** grave uma transação de `Tipo=TransferenciaSaida` ou `TransferenciaEntrada` fora do fluxo de `Transferencia.Registrar` — é assim que a regra "toda transferência nasce em par" é garantida (ver `Transacao.CriarPernaTransferencia`, `internal`, no diagrama de classes).

---

## Transferencia

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdTransferencia | int | não | PK | autoincremento | |
| IdContaOrigem | int | não | FK → Conta.IdConta | | Debitada |
| IdContaDestino | int | não | FK → Conta.IdConta | ≠ IdContaOrigem | Creditada — validar que origem ≠ destino |
| Data | date | não | | | |
| Valor | decimal(18,2) | não | | `> 0` | Valor igual nas duas pernas — este domínio não modela taxa de transferência |
| Descricao | string(255) | sim | | livre | |

Excluir uma `Transferencia` exclui as duas `Transacao` ligadas a ela na mesma operação — nunca deixe uma perna órfã.

---

## Orcamento

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdOrcamento | int | não | PK | autoincremento | |
| IdCategoria | int | não | FK → Categoria.IdCategoria | `Categoria.Tipo = Despesa` | Orçamento não existe para categoria de receita |
| MesReferencia | date | não | UK composta com IdCategoria | primeiro dia do mês (ex.: `2026-09-01`) | Um orçamento por categoria por mês — inserir duplicata é erro de aplicação, não upsert silencioso |
| ValorMeta | decimal(18,2) | não | | `> 0` | Teto de gasto do mês. Estourar **gera alerta, nunca bloqueia** o lançamento da despesa |

Sem rollover: o consumo reinicia zerado todo mês, mesmo que o mês anterior tenha sobrado orçamento.

---

## MetaEconomia

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdMeta | int | não | PK | autoincremento | |
| IdConta | int | sim | FK → Conta.IdConta | | Conta onde o valor da meta fica reservado, se houver uma dedicada |
| Nome | string(100) | não | | livre | |
| ValorAlvo | decimal(18,2) | não | | `> 0` | |
| DataAlvo | date | sim | | | Data desejada para atingir o alvo — informativa, não gera bloqueio |
| Concluida | bool | não | | `true`/`false` | Marcada quando `TotalAportado() >= ValorAlvo`; é só um estado de exibição |

## AporteMeta

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdAporte | int | não | PK | autoincremento | |
| IdMeta | int | não | FK → MetaEconomia.IdMeta | | |
| IdTransacao | int | sim | FK → Transacao.IdTransacao | | Transação/transferência que originou o aporte, se houver uma associada diretamente |
| Data | date | não | | | |
| Valor | decimal(18,2) | não | | `> 0` | A meta não valida saldo — quem bloqueia (ou não) é a regra de saldo da conta de origem do aporte |

---

## Cartao

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdCartao | int | não | PK | autoincremento | |
| IdContaPagamento | int | não | FK → Conta.IdConta | | Conta debitada quando a fatura é paga |
| Nome | string(100) | não | | livre | |
| LimiteTotal | decimal(18,2) | não | | `> 0` | |
| DiaFechamento | int | não | | `1`–`31` | Se o mês não tiver esse dia, usar o último dia do mês |
| DiaVencimento | int | não | | `1`–`31` | |
| Ativo | bool | não | | `true`/`false` | |

**Cartão não tem saldo.** Não crie uma coluna de saldo nesta tabela — o estado financeiro do cartão é sempre derivado (`LimiteDisponivel`), nunca armazenado como um número que pode divergir da soma real.

---

## Fatura

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdFatura | int | não | PK | autoincremento | |
| IdCartao | int | não | FK → Cartao.IdCartao | | |
| IdTransacaoPagamento | int | sim | FK → Transacao.IdTransacao | | Preenchido só quando `Status=Paga` |
| MesReferencia | date | não | UK composta com IdCartao | primeiro dia do mês do ciclo | Uma fatura por cartão por mês |
| DataFechamento | date | não | | | |
| DataVencimento | date | não | | | |
| ValorTotal | decimal(18,2) | não | | `≥ 0` | Denormalizado: soma de `Parcela.Valor` das parcelas dessa fatura. Recalcular a cada escrita em `Parcela` |
| Status | int (enum) | não | | `Aberta=0`, `Fechada=1`, `Paga=2` | Fatura `Fechada`/`Paga` é imutável — nenhuma parcela nova entra nela |

---

## Compra

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdCompra | int | não | PK | autoincremento | |
| IdCartao | int | não | FK → Cartao.IdCartao | | |
| IdCategoria | int | não | FK → Categoria.IdCategoria | `Categoria.Tipo = Despesa` | |
| IdRecorrencia | int | sim | FK → Recorrencia.IdRecorrencia | | Preenchida quando gerada automaticamente (assinatura no cartão) |
| Data | date | não | | | Define em qual ciclo (fatura) a 1ª parcela cai, conforme `DiaFechamento` do cartão |
| Descricao | string(255) | não | | livre | |
| ValorTotal | decimal(18,2) | não | | `> 0` | Soma de todas as parcelas |
| NumeroParcelas | int | não | | `≥ 1` | |

**Antes de gravar**: `ValorTotal` precisa ser `≤ Cartao.LimiteDisponivel()` (soma de todas as parcelas futuras não pagas de todas as faturas). Bloquear a compra inteira se exceder — não é permitido gravar parte das parcelas.

## Parcela

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdParcela | int | não | PK | autoincremento | |
| IdCompra | int | não | FK → Compra.IdCompra | | |
| IdFatura | int | não | FK → Fatura.IdFatura | | Cada parcela cai em uma fatura consecutiva a partir da fatura da compra |
| Numero | int | não | | `1`..`NumeroParcelas` | |
| Valor | decimal(18,2) | não | | `> 0` | `ValorTotal / NumeroParcelas`, com a última parcela ajustada para a soma bater exatamente (arredondamento de centavos) |

---

## Recorrencia

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| IdRecorrencia | int | não | PK | autoincremento | |
| IdConta | int | sim | FK → Conta.IdConta | mutuamente exclusivo com IdCartao | Preenchido quando gera `Transacao` (receita/despesa em conta) |
| IdCartao | int | sim | FK → Cartao.IdCartao | mutuamente exclusivo com IdConta | Preenchido quando gera `Compra` (cartão) |
| IdCategoria | int | não | FK → Categoria.IdCategoria | | |
| TipoTransacao | int (enum) | sim | | `Receita=0`, `Despesa=1` | Só usado quando `IdConta` preenchido |
| Valor | decimal(18,2) | não | | `> 0` | |
| Frequencia | int (enum) | não | | `Semanal=0`, `Mensal=1`, `Anual=2` | |
| DiaGeracao | int | não | | `1`–`31` (mensal/anual) ou `0`–`6` (semanal) | Se o dia não existir no mês, gera no último dia do mês |
| DataInicio | date | não | | | |
| DataFim | date | sim | | | Sem data fim = recorre indefinidamente |
| Ativa | bool | não | | `true`/`false` | Pausar/cancelar impede geração futura, mas não apaga nem reverte transações já geradas |

**Constraint de aplicação** (não dá para expressar como `CHECK` simples de forma legível — validar em `Recorrencia.Criar`): exatamente um entre `IdConta` e `IdCartao` deve estar preenchido, nunca os dois, nunca nenhum.

---

## ApplicationUser

| Campo | Tipo | Nulo | Chave | Domínio / Valores válidos | Descrição / Regra |
|---|---|---|---|---|---|
| Id | string | não | PK | GUID em texto (padrão Identity) | |
| Nome | string(100) | não | | livre | Campo próprio do domínio, além do que o Identity já traz |
| Email | string | não | UK | e-mail válido | Vem do `IdentityUser` — login, hash de senha etc. são responsabilidade do `UserManager`, não desta tabela |

Todas as tabelas de negócio acima referenciam `ApplicationUser.Id` através da coluna implícita `IdUsuario` (ver convenções no topo).
