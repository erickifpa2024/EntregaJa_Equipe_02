# ADR-002 — Publicação de eventos de domínio para mudanças de status

- **Status:** Aceito
- **Data:** 2026-10-28

## Contexto

Quando o status do pedido muda (ex.: PAGO → EM_PREPARO), múltiplos módulos precisam reagir: módulo de entrega, módulo de notificação e módulo de avaliação. Chamadas síncronas diretas entre esses módulos criariam acoplamento interno que dificulta testes e extração futura.

## Decisão

`PedidoService` publica eventos de domínio internos (in-process) ao alterar o status do pedido. Os módulos interessados registram handlers para esses eventos. Em fase futura, os mesmos eventos podem ser externalizados para um broker (ex.: RabbitMQ ou Kafka) sem alterar o núcleo do domínio.

## Alternativas consideradas

- **Chamadas síncronas diretas entre serviços internos:** descartado porque cria dependências diretas entre módulos, tornando mais difícil testar cada um de forma isolada e dificultar extração futura.
- **Mensageria externa (RabbitMQ/Kafka) desde o início:** descartado porque o monólito modular não exige infraestrutura de broker nesta fase, aumentando complexidade operacional sem ganho real neste estágio.

## Consequências

- **Positivas:** Menor acoplamento entre módulos; facilita extração futura; alinhado com RNF07 (integrabilidade).
- **Negativas/trade-offs:** Requer atenção à idempotência dos handlers (RN04) e à ordem de processamento dos eventos. Consistência eventual pode ocorrer se um handler falhar após o evento ser publicado.
