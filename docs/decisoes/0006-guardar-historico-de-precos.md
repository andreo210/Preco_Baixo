# 0006 — Guardar histórico com delta + downsampling

**Status:** aceita · **Data:** 2026-09-30

## Contexto
O histórico de preço é **ativo estratégico** (alimenta detecção de queda, gráfico do site e fases futuras), mas salvar toda verificação faria o banco crescer rápido. A coleta roda 24h.

## Decisão
1. Gravar `PrecoHistorico` **só quando o preço muda** (por variação) + 1 "heartbeat" diário.
2. Manter resolução cheia por **~90 dias**.
3. **Downsamplear** diariamente em `PrecoDiario` (min/max/fechamento) e **remover** o `PrecoHistorico` além de 90 dias.
4. **Nunca** apagar `PrecoDiario` (barato: 1 linha/produto/dia).
5. Escala: particionar `PrecoHistorico` por mês.
6. Preço sempre em **inteiro (centavos)**.

## Alternativas
- **Salvar toda checagem:** simples, mas ~1,7 GB/ano por 1.000 produtos e cresce linear com a frequência. Desnecessário.
- **Só preço atual (sem histórico):** mataria o diferencial ("queda verificada") e o gráfico.

## Consequências
- Banco cresce devagar e previsível (dezenas de MB/ano no MVP).
- Gráfico do site lê `PrecoDiario` (rápido).
- Cálculo de queda usa a série recente por variação (ver `docs/05`).
- Detalhes em `docs/04`.
