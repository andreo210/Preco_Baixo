# Diagramas de sequência

Seis fluxos escolhidos porque cada um materializa um dos "invariantes críticos" da skill `regras-negocio-financas` — não é uma cobertura exaustiva de todo endpoint, é o comportamento que mais fácil de implementar errado. Os nomes de método (`InserirSalvarAsync`, `ExecuteTransactionAsync`, `_notificador.Add`, `CustomResponse`) são os reais da arquitetura já definida em `arquitetura-api` — copie o padrão, não invente um novo.

## 1. Registrar despesa — bloqueio por saldo vs. alerta de orçamento

O caso mais importante de diferenciar: **saldo insuficiente bloqueia** (quando a conta não permite negativar), **orçamento estourado só alerta**. São dois mecanismos diferentes de propósito porque um response de erro (400) não pode carregar ao mesmo tempo "sua despesa foi rejeitada" e "sua despesa foi aceita, mas passou do orçamento" — por isso o alerta de orçamento **não** passa pelo `INotificadorService` (que sempre vira erro 4xx); ele viaja como campo informativo no DTO de retorno de uma resposta 201.

```mermaid
sequenceDiagram
    actor Cliente as Cliente (Front)
    participant Controller as TransacaoController
    participant Service as TransacaoService
    participant Notificador as INotificadorService
    participant ContaRepo as ContaRepository
    participant Conta as Conta (entidade)
    participant Orcamento as OrcamentoService
    participant TransacaoRepo as TransacaoRepository
    participant Banco

    Cliente->>Controller: POST /transacoes (despesa)
    Controller->>Service: RegistrarDespesaAsync(dto)
    Service->>ContaRepo: ObterPorIdAsync(dto.IdConta)
    ContaRepo-->>Service: conta
    Service->>Conta: PermiteSaldoNegativo()
    Conta-->>Service: bool
    Service->>Service: saldoResultante = SaldoAtual(conta) - dto.Valor

    alt saldoResultante < 0 e PermiteSaldoNegativo() == false
        Service->>Notificador: Add("Saldo insuficiente")
        Service-->>Controller: null
        Controller-->>Cliente: 400 ProblemDetails (via CustomResponse)
    else saldo permitido (ou fica negativo em conta Corrente)
        Service->>Transacao: RegistrarDespesa(idConta, idCategoria, data, valor, descricao)
        Transacao-->>Service: transacao
        Service->>TransacaoRepo: InserirSalvarAsync(transacao)
        TransacaoRepo->>Banco: INSERT
        Banco-->>TransacaoRepo: ok
        Service->>Orcamento: VerificarEstouro(idCategoria, mesReferencia)
        Note right of Orcamento: consulta o Orcamento do mes e soma o gasto\nda categoria — nao usa o Notificador aqui:\nisso faria a resposta virar 400 e bloquearia\num lancamento que a regra manda aceitar
        Orcamento-->>Service: alertaOrcamento (string?, null se dentro do teto)
        Service-->>Controller: TransacaoDto { ..., AlertaOrcamento }
        Controller-->>Cliente: 201 Created (com AlertaOrcamento preenchido se estourou)
    end
```

## 2. Transferir entre contas — atomicidade das duas pernas

`Transferencia.Registrar` nunca é chamado fora de uma transação de banco — se a perna de entrada falhasse depois da de saída já ter sido persistida, o dinheiro "sumiria" de uma conta sem aparecer na outra. É exatamente para isso que `IUnitOfWork.ExecuteTransactionAsync` existe (ver `persistencia.md` do `arquitetura-api`): abre, roda as duas inserções sem salvar (`InserirAsync`), e só comita as duas juntas no fim.

```mermaid
sequenceDiagram
    actor Cliente as Cliente (Front)
    participant Controller as TransferenciaController
    participant Service as TransferenciaService
    participant Notificador as INotificadorService
    participant ContaRepo as ContaRepository
    participant UoW as IUnitOfWork
    participant TransfRepo as TransferenciaRepository
    participant TransacaoRepo as TransacaoRepository
    participant Banco

    Cliente->>Controller: POST /transferencias
    Controller->>Service: RegistrarAsync(dto)
    Service->>ContaRepo: ObterPorIdAsync(dto.IdContaOrigem)
    ContaRepo-->>Service: contaOrigem
    Service->>Service: valida saldo da origem (mesma regra da despesa)

    alt saldo insuficiente na origem
        Service->>Notificador: Add("Saldo insuficiente na conta de origem")
        Service-->>Controller: null
        Controller-->>Cliente: 400 ProblemDetails
    else saldo ok
        Service->>UoW: ExecuteTransactionAsync(async () => { ... })
        activate UoW
        UoW->>Transferencia: Registrar(idOrigem, idDestino, data, valor, descricao)
        Note right of Transferencia: internamente cria as duas pernas via\nTransacao.CriarPernaTransferencia (internal) —\nninguem fora daqui cria essas transacoes
        Transferencia-->>UoW: transferencia (com as 2 Transacao em memoria)
        UoW->>TransfRepo: InserirAsync(transferencia)
        UoW->>TransacaoRepo: InserirAsync(transacaoSaida)
        UoW->>TransacaoRepo: InserirAsync(transacaoEntrada)
        UoW->>Banco: SaveChanges + COMMIT
        alt qualquer insert falha
            UoW->>Banco: ROLLBACK
            UoW-->>Service: throw
            Service-->>Controller: propaga (middleware converte em 500/ProblemDetails)
        else tudo ok
            Banco-->>UoW: ok
            deactivate UoW
            UoW-->>Service: transferencia
            Service-->>Controller: TransferenciaDto
            Controller-->>Cliente: 201 Created
        end
    end
```

## 3. Registrar compra parcelada no cartão — limite disponível

O ponto que mais gera bug: o limite disponível considera **todas** as parcelas futuras não pagas, não só a fatura aberta — então a validação precisa somar parcelas de várias faturas antes de decidir. Se aprovado, a compra gera N parcelas distribuídas em N faturas consecutivas (abrindo fatura nova quando ainda não existir uma para aquele mês).

```mermaid
sequenceDiagram
    actor Cliente as Cliente (Front)
    participant Controller as CompraController
    participant Service as CompraService
    participant Notificador as INotificadorService
    participant CartaoRepo as CartaoRepository
    participant Cartao as Cartao (entidade)
    participant ParcelaRepo as ParcelaRepository
    participant FaturaRepo as FaturaRepository
    participant Banco

    Cliente->>Controller: POST /compras (valor total, numeroParcelas)
    Controller->>Service: RegistrarAsync(dto)
    Service->>CartaoRepo: ObterPorIdAsync(dto.IdCartao)
    CartaoRepo-->>Service: cartao
    Service->>ParcelaRepo: SomarParcelasFuturasNaoPagasAsync(idCartao)
    ParcelaRepo-->>Service: totalComprometido
    Service->>Cartao: LimiteDisponivel(totalComprometido)
    Cartao-->>Service: limiteDisponivel

    alt dto.ValorTotal > limiteDisponivel
        Service->>Notificador: Add("Limite disponível insuficiente")
        Service-->>Controller: null
        Controller-->>Cliente: 400 ProblemDetails
    else dentro do limite
        Service->>Compra: Registrar(idCartao, idCategoria, data, descricao, valorTotal, numeroParcelas)
        Compra-->>Service: compra
        loop para cada parcela de 1 a numeroParcelas
            Service->>FaturaRepo: ObterOuAbrirFaturaDoCiclo(idCartao, mesDaParcela)
            Note right of FaturaRepo: se a fatura do mes ainda nao existe, abre uma\nnova (Fatura.Abrir) — se ja existir e estiver\nFechada/Paga, é erro de calculo de ciclo, nao um\ncaminho valido aqui
            FaturaRepo-->>Service: fatura
            Service->>Parcela: Criar(idCompra, fatura.IdFatura, numero, valorDaParcela)
            Parcela-->>Service: parcela
            Service->>ParcelaRepo: InserirAsync(parcela)
            Service->>Fatura: RecalcularValorTotal(somaParcelas)
        end
        Service->>Banco: SaveChangesAsync (compra + parcelas + faturas atualizadas)
        Banco-->>Service: ok
        Service-->>Controller: CompraDto
        Controller-->>Cliente: 201 Created
    end
```

## 4. Job — fechamento automático da fatura

Roda diariamente (hosted service). Para cada cartão cujo `DiaFechamento` é hoje, fecha a fatura do ciclo atual (trava novos lançamentos nela) e garante que a fatura do próximo ciclo já exista aberta, para que compras feitas amanhã tenham onde cair.

```mermaid
sequenceDiagram
    participant Job as FechamentoFaturaJob
    participant Scope as IServiceScopeFactory
    participant CartaoRepo as CartaoRepository
    participant FaturaRepo as FaturaRepository
    participant Fatura as Fatura (entidade)
    participant Banco

    Job->>Job: dispara todo dia à meia-noite (hosted service)
    Job->>Scope: CreateScope()
    Note right of Scope: job é singleton — precisa abrir escopo\npara resolver repositorio/DbContext scoped
    Scope-->>Job: escopo com CartaoRepository, FaturaRepository

    Job->>CartaoRepo: ObterAsync(c => c.DiaFechamento == hoje.Day && c.Ativo)
    CartaoRepo-->>Job: cartoesDoDia

    loop para cada cartão
        Job->>FaturaRepo: ObterPrimeiroAsync(f => f.IdCartao == cartao.IdCartao && f.Status == Aberta, rastreado: true)
        FaturaRepo-->>Job: faturaAtual
        Job->>Fatura: Fechar()
        Fatura-->>Job: status = Fechada
        Job->>FaturaRepo: AtualizarSalvarAsync(faturaAtual)

        Job->>FaturaRepo: ExisteAsync(f => f.IdCartao == cartao.IdCartao && f.MesReferencia == proximoMes)
        alt fatura do próximo ciclo ainda não existe
            Job->>Fatura: Abrir(cartao.IdCartao, proximoMes, proximoFechamento, proximoVencimento)
            Fatura-->>Job: novaFatura
            Job->>FaturaRepo: InserirSalvarAsync(novaFatura)
        end
    end
    Job->>Banco: (cada AtualizarSalvarAsync/InserirSalvarAsync já salva)
```

## 5. Pagar fatura — gera despesa e libera limite

Pagar a fatura não é uma entidade nova nem um valor guardado à parte: é uma despesa comum na conta de pagamento do cartão. O limite volta a ficar disponível como efeito colateral de `Status = Paga` — nenhuma parcela é apagada, só deixa de contar como "não paga" na soma que `LimiteDisponivel` usa.

```mermaid
sequenceDiagram
    actor Cliente as Cliente (Front)
    participant Controller as FaturaController
    participant Service as FaturaService
    participant Notificador as INotificadorService
    participant FaturaRepo as FaturaRepository
    participant Fatura as Fatura (entidade)
    participant ContaRepo as ContaRepository
    participant TransacaoRepo as TransacaoRepository
    participant Banco

    Cliente->>Controller: POST /faturas/{id}/pagamento
    Controller->>Service: RegistrarPagamentoAsync(idFatura)
    Service->>FaturaRepo: ObterPorIdAsync(idFatura, rastreado: true)
    FaturaRepo-->>Service: fatura

    alt fatura.Status == Paga
        Service->>Notificador: Add("Fatura já está paga")
        Service-->>Controller: null
        Controller-->>Cliente: 400 ProblemDetails
    else fatura Aberta ou Fechada
        Service->>ContaRepo: ObterPorIdAsync(cartao.IdContaPagamento)
        ContaRepo-->>Service: contaPagamento
        Service->>Service: valida saldo da contaPagamento (mesma regra da despesa)
        alt saldo insuficiente
            Service->>Notificador: Add("Saldo insuficiente na conta de pagamento")
            Service-->>Controller: null
            Controller-->>Cliente: 400 ProblemDetails
        else saldo ok
            Service->>Transacao: RegistrarDespesa(idContaPagamento, categoriaPagamentoFatura, hoje, fatura.ValorTotal, "Pagamento fatura")
            Transacao-->>Service: transacaoPagamento
            Service->>TransacaoRepo: InserirSalvarAsync(transacaoPagamento)
            Service->>Fatura: RegistrarPagamento(transacaoPagamento.IdTransacao)
            Fatura-->>Service: status = Paga
            Service->>FaturaRepo: AtualizarSalvarAsync(fatura)
            FaturaRepo->>Banco: UPDATE
            Banco-->>Service: ok
            Service-->>Controller: FaturaDto
            Controller-->>Cliente: 200 OK
        end
    end
```

## 6. Job — geração de ocorrência de recorrência

Roda diariamente. Para cada `Recorrencia` ativa cuja próxima data de geração é hoje, cria a transação (receita/despesa) ou a compra correspondente, na conta ou cartão configurado — e é exatamente o mesmo caminho de "Registrar despesa"/"Registrar compra" usado quando um humano lança manualmente, incluindo a validação de saldo/limite. Uma recorrência não tem passe livre para violar o mesmo invariante que bloquearia um lançamento manual.

```mermaid
sequenceDiagram
    participant Job as RecorrenciaJob
    participant Scope as IServiceScopeFactory
    participant RecorrenciaRepo as RecorrenciaRepository
    participant Recorrencia as Recorrencia (entidade)
    participant TransacaoService
    participant CompraService
    participant Notificador as INotificadorService
    participant Log

    Job->>Job: dispara todo dia à meia-noite (hosted service)
    Job->>Scope: CreateScope()
    Scope-->>Job: escopo com RecorrenciaRepository, TransacaoService, CompraService

    Job->>RecorrenciaRepo: ObterAsync(r => r.Ativa)
    RecorrenciaRepo-->>Job: recorrenciasAtivas

    loop para cada recorrência
        Job->>Recorrencia: ProximaDataGeracao(hoje)
        Recorrencia-->>Job: proximaData
        alt proximaData == hoje
            alt Recorrencia.IdConta preenchido
                Job->>TransacaoService: RegistrarReceitaOuDespesaAsync(idConta, idCategoria, valor, tipo, origem: recorrencia)
                Note right of TransacaoService: mesmo fluxo do diagrama 1 — se a conta\nnao permite saldo negativo e ficaria negativa,\na geracao desta ocorrencia é bloqueada
                TransacaoService-->>Job: sucesso ou notificação de bloqueio
            else Recorrencia.IdCartao preenchido
                Job->>CompraService: RegistrarAsync(idCartao, idCategoria, valor, numeroParcelas: 1, origem: recorrencia)
                Note right of CompraService: mesmo fluxo do diagrama 3 — se o limite\ndisponivel nao comporta, a geracao desta\nocorrencia é bloqueada
                CompraService-->>Job: sucesso ou notificação de bloqueio
            end
            alt geração bloqueada (saldo/limite insuficiente)
                Job->>Log: registra falha de geração da recorrência X para revisão do usuário
            else geração ok
                Job->>Recorrencia: nada a fazer aqui — próxima data recalculada\na partir de DiaGeracao/Frequencia na próxima consulta
            end
        end
    end
```
