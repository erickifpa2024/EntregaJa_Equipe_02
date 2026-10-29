# ADR-001 — Reserva de estoque no momento do checkout

- **Status:** Aceito
- **Data:** 2026-10-28

## Contexto

Dois clientes podem tentar comprar o último item ao mesmo tempo. Se o estoque for verificado antes e decrementado depois em operações separadas, existe uma janela de concorrência que resulta em estoque negativo ou venda duplicada. O requisito RN01 do guia especifica que o estoque deve ser validado antes de confirmar o pedido.

## Decisão

Reservar o estoque dentro da mesma transação do checkout, usando bloqueio otimista (`@Version` no JPA) na entidade `Estoque`. Em caso de conflito de versão, a transação perdedora recebe erro 409 e o cliente deve tentar novamente.

## Alternativas consideradas

- **Reservar após confirmação do pagamento:** descartado porque o pagamento pode ser confirmado para um estoque já esgotado, gerando cancelamento posterior e experiência ruim para o cliente.
- **Saga distribuída desde o início:** descartado porque o projeto está em monólito modular nesta fase; saga é complexidade operacional desnecessária antes do Projeto Final.

## Consequências

- **Positivas:** Maior consistência transacional; elimina a possibilidade de estoque negativo.
- **Negativas/trade-offs:** Possível contenção sob pico de acessos ao mesmo produto, mitigada pelo retry no cliente. Requer que `EstoqueService` e `CheckoutService` operem dentro da mesma transação.
