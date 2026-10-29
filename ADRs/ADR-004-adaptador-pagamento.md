# ADR-004 — Adaptador de pagamento como interface abstrata

- **Status:** Aceito
- **Data:** 2026-10-28

## Contexto

O gateway de pagamento é externo e simulado nesta fase. RNF07 exige que trocar o gateway não altere o núcleo do domínio de pedidos. Pagamentos exigem idempotência (RN04) para evitar cobranças duplicadas em caso de reenvio ou retry da requisição.

## Decisão

Definir a interface `GatewayPagamento` no módulo de Pagamento. `PagamentoAdapter` implementa essa interface apontando para o gateway simulado. `PagamentoService` depende apenas da interface, nunca da implementação concreta. A chave de idempotência é gerada por `PagamentoService` antes de chamar o adaptador, garantindo que reenvios da mesma operação sejam tratados como duplicata.

## Alternativas consideradas

- **Chamar o gateway diretamente no `PedidoService`:** descartado porque acopla o domínio de pedidos a um fornecedor externo específico e mistura responsabilidades de domínio com integração.
- **Acoplamento fixo a um único provedor sem interface:** descartado pelo mesmo motivo; viola RNF07 diretamente e torna o teste unitário dependente de infraestrutura externa.

## Consequências

- **Positivas:** Facilita troca de gateway sem alterar `PedidoService`; permite mock da interface em testes unitários; encapsula a lógica de idempotência em um único lugar.
- **Negativas/trade-offs:** Exige definição e manutenção de um contrato claro na interface `GatewayPagamento`. Adiciona uma camada de indireção que deve ser mantida coerente com as evoluções do domínio de pagamento.
