# 0003 — Loja e canal de notificação como adaptadores plugáveis

**Status:** aceita · **Data:** 2026-09-30

## Contexto
O roadmap prevê novas lojas (Amazon, Mercado Livre) e novos canais (WhatsApp, e-mail) sem reescrever o sistema. O motor (coleta/histórico/detecção) é comum; muda só a origem do preço e o destino da mensagem.

## Decisão
Definir **duas interfaces de adaptador**:
- **Adaptador de loja** (`/adapters`): `buscar_produto(link)`, `consultar_preco(produto)`, `gerar_link_afiliado(produto, sub_id)`.
- **Adaptador de canal** (`/channels`): `publicar(publicacao)` (e, na fase 2, `enviar_dm(usuario, mensagem)`).

O núcleo depende só das interfaces, nunca de uma loja/canal concreto.

## Alternativas
- Código acoplado a Shopee/Telegram diretamente: mais rápido no início, caro para expandir. Descartado dado o roadmap multi-loja/multi-canal.

## Consequências
- Adicionar loja ou canal = implementar uma interface, sem tocar no motor (RNF-04, RNF-12).
- Facilita testes (adaptadores fake).
- Pequeno custo inicial de abstração, já justificado pelo roadmap.
