# ADR-005 — Cache seletivo para consultas do catálogo

- **Status:** Aceito
- **Data:** 2026-10-28

## Contexto

RNF01 e RNF02 exigem latência abaixo de 2s no p95 mesmo com pico de acesso 10× no catálogo (ex.: promoções). Consultas de produto e categoria são read-heavy e relativamente estáveis. Estoque e preço no momento do checkout, por outro lado, precisam sempre refletir o valor atual para garantir consistência (RN01 e RN03).

## Decisão

Aplicar cache em memória (Spring Cache + Caffeine) nas operações de leitura de `CatalogoService`: listagem de categorias e busca de produtos por filtro. TTL configurável externamente. Estoque e preço no checkout são consultados sempre direto no banco, sem cache — o catálogo pode exibir dado levemente desatualizado, mas o checkout valida a fonte de verdade.

## Alternativas consideradas

- **Consultar banco em toda requisição de catálogo:** descartado porque não atende RNF01/RNF02 sob carga de promoção sem escalabilidade horizontal, que não está prevista para esta fase.
- **Cachear tudo, incluindo estoque e preço no checkout:** descartado porque estoque pode mudar entre a exibição no catálogo e a finalização da compra — usar dado em cache como fonte definitiva causaria vendas sem estoque real disponível, violando RN01.

## Consequências

- **Positivas:** Reduz latência e carga no banco para operações de leitura do catálogo; alinhado com RNF01 e RNF02.
- **Negativas/trade-offs:** Risco de catálogo exibir dado levemente desatualizado (aceitável, pois checkout valida em tempo real). Requer definição de TTL adequado e estratégia de invalidação ao atualizar produtos/categorias no painel administrativo.
