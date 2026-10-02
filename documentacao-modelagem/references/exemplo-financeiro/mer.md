# MER — Modelo Entidade-Relacionamento (conceitual)

Visão de negócio do domínio do BolsoEmDia: só entidades, seus atributos-chave e como se relacionam. Sem tipos de dado, sem chave estrangeira explícita — isso fica no [DER](der.md). Se uma regra aqui parecer estranha, ela vem de `regras-negocio-financas` (skill do repositório) — este diagrama é a tradução visual daquelas regras, não uma fonte nova.

```mermaid
erDiagram
    APPLICATION_USER ||--o{ CONTA : possui
    APPLICATION_USER ||--o{ CATEGORIA : possui
    APPLICATION_USER ||--o{ CARTAO : possui
    APPLICATION_USER ||--o{ ORCAMENTO : define
    APPLICATION_USER ||--o{ META_ECONOMIA : define
    APPLICATION_USER ||--o{ RECORRENCIA : define

    CONTA ||--o{ TRANSACAO : movimenta
    CONTA ||--o{ TRANSFERENCIA : "é origem de"
    CONTA ||--o{ TRANSFERENCIA : "é destino de"
    CONTA ||--o{ CARTAO : "recebe pagamento de"
    CONTA |o--o{ META_ECONOMIA : reserva

    CATEGORIA ||--o{ TRANSACAO : classifica
    CATEGORIA ||--o{ ORCAMENTO : limita
    CATEGORIA ||--o{ COMPRA : classifica
    CATEGORIA ||--o{ RECORRENCIA : classifica
    CATEGORIA |o--o{ CATEGORIA : "é subcategoria de"

    TRANSFERENCIA ||--|{ TRANSACAO : "gera (saida + entrada)"

    CARTAO ||--o{ FATURA : gera
    CARTAO ||--o{ COMPRA : recebe

    FATURA ||--o{ PARCELA : agrupa
    FATURA |o--|| TRANSACAO : "é paga por"

    COMPRA ||--|{ PARCELA : divide

    META_ECONOMIA ||--o{ APORTE_META : recebe
    APORTE_META |o--|| TRANSACAO : origina

    RECORRENCIA |o--o{ TRANSACAO : "gera (receita/despesa)"
    RECORRENCIA |o--o{ COMPRA : "gera (compra no cartao)"

    APPLICATION_USER {
        string Id PK
        string Nome
        string Email
    }
    CONTA {
        int IdConta PK
        string Tipo
        string Nome
    }
    CATEGORIA {
        int IdCategoria PK
        string Tipo
        string Nome
    }
    TRANSACAO {
        int IdTransacao PK
        string Tipo
        decimal Valor
        date Data
    }
    TRANSFERENCIA {
        int IdTransferencia PK
        decimal Valor
        date Data
    }
    ORCAMENTO {
        int IdOrcamento PK
        date MesReferencia
        decimal ValorMeta
    }
    META_ECONOMIA {
        int IdMeta PK
        string Nome
        decimal ValorAlvo
    }
    CARTAO {
        int IdCartao PK
        string Nome
        decimal LimiteTotal
    }
    FATURA {
        int IdFatura PK
        date MesReferencia
        string Status
    }
    COMPRA {
        int IdCompra PK
        decimal ValorTotal
        int NumeroParcelas
    }
    PARCELA {
        int IdParcela PK
        int Numero
        decimal Valor
    }
    APORTE_META {
        int IdAporte PK
        decimal Valor
        date Data
    }
    RECORRENCIA {
        int IdRecorrencia PK
        string Frequencia
        decimal Valor
    }
```

## Leitura rápida das relações que fogem do óbvio

- **Cartão não empresta saldo de conta** — a única ponte entre `CARTAO` e `CONTA` é "recebe pagamento de": a conta só entra quando a fatura é paga.
- **Transferência gera duas transações**, nunca uma só — por isso a cardinalidade é `||--|{` (uma transferência, uma-ou-mais... na prática sempre exatas duas) e não `||--o{`.
- **Compra e Transação não se tocam.** Uma compra no cartão nunca vira uma linha em `TRANSACAO` diretamente — ela move para `CONTA` só indiretamente, quando a `FATURA` que a contém é paga (aí sim nasce uma `TRANSACAO` de pagamento).
- **Categoria se relaciona consigo mesma** (subcategoria de) — hierarquia de um nível, ver `dicionario-dados.md` para a regra de agregação de totais.
- **Recorrência é a origem de duas coisas diferentes**: transações (receita/despesa em conta) ou compras (cartão), nunca as duas ao mesmo tempo para o mesmo registro.
