# Matriz de Rastreabilidade — EntregaJá

> **Nota:** Os identificadores de requisitos seguem o Guia Detalhado do Estudo de Caso (RF01–RF12, RN01–RN07, RNF01–RNF07). As ADRs referenciadas estão em `docs/adr/`.

---

## Requisitos Funcionais

| Requisito | Descrição | ADR relacionada | Componente / Tática | Como verificar |
|-----------|-----------|-----------------|---------------------|----------------|
| RF01 | Manter catálogo de produtos | ADR-003, ADR-005 | `CatalogoService`, cache seletivo | CRUD de produtos; medir latência de listagem |
| RF02 | Buscar e filtrar produtos | ADR-003, ADR-005 | `CatalogoService`, paginação | Pesquisa por nome/categoria; latência < 1,5 s no p95 |
| RF03 | Gerenciar categorias | ADR-003 | `CatalogoService` | CRUD de categorias; retorno correto por ID |
| RF04 | Montar e editar carrinho | ADR-003 | `CarrinhoService`, `ItemDoCarrinho` | Adicionar/remover itens; persistência entre sessões |
| RF05 | Finalizar compra (checkout) | ADR-001, ADR-004 | `CheckoutService`, `EstoqueService` | Checkout com reserva de estoque; teste de concorrência na última unidade |
| RF06 | Processar pagamento | ADR-004 | `PagamentoService`, `PagamentoAdapter` | Simular aprovação e recusa do gateway simulado |
| RF07 | Garantir idempotência de pagamento | ADR-004 | `PagamentoService`, chave de idempotência | Reenviar mesma chave; verificar cobrança única |
| RF08 | Atualizar e consultar status do pedido | ADR-002 | `PedidoService`, eventos de domínio | Alterar status; consultar histórico de estados |
| RF09 | Acompanhar entrega | ADR-002 | `EntregaService`, eventos de domínio | Atribuir entregador; atualizar e consultar status de entrega |
| RF10 | Gerenciar estoque | ADR-001 | `EstoqueService`, bloqueio otimista (`@Version`) | Compra simultânea da última unidade → estoque nunca negativo |
| RF11 | Controle de acesso por perfil | ADR-006 | Spring Security, RBAC (`ROLE_LOJISTA`, `ROLE_ADMIN`) | Testar acesso permitido e negado por perfil (cliente, lojista, entregador, admin) |
| RF12 | Avaliar pedido entregue | ADR-002 | `AvaliacaoService`, evento `PEDIDO_ENTREGUE` | Avaliação só disponível após status ENTREGUE (RN05) |

---

## Regras de Negócio

| Requisito | Descrição | ADR relacionada | Componente / Tática | Como verificar |
|-----------|-----------|-----------------|---------------------|----------------|
| RN01 | Validar estoque antes de confirmar pedido | ADR-001 | `EstoqueService`, transação atômica | Dois usuários compram última unidade; apenas um deve ser confirmado |
| RN02 | Iniciar preparo somente após pagamento | — | Máquina de estados de `Pedido` | Tentar forçar transição para EM_PREPARO sem pagamento → rejeição |
| RN03 | Preço registrado no momento da compra | ADR-003 | `ItemDoPedido.precoUnitario` (imutável) | Alterar preço do produto; verificar que pedido existente mantém valor original |
| RN04 | Idempotência no pagamento | ADR-004 | `PagamentoService`, chave de idempotência | Reprocessar evento de pagamento; estado deve permanecer o mesmo |
| RN05 | Avaliação somente após entrega | ADR-002 | Máquina de estados, `AvaliacaoService` | Tentar avaliar pedido EM_ROTA → erro esperado |
| RN06 | Status do pedido não retrocede | — | Máquina de estados de `Pedido` | Tentar regredir PAGO → CRIADO → rejeição |
| RN07 | Cancelamento segue regras explícitas | — | Máquina de estados, transições de cancelamento | Cancelar nas fases permitidas; verificar bloqueio fora delas |

---

## Requisitos Não Funcionais

| Requisito | Descrição | ADR relacionada | Componente / Tática | Como verificar |
|-----------|-----------|-----------------|---------------------|----------------|
| RNF01 | Escalabilidade do catálogo (pico 10×) | ADR-005 | `CatalogoService`, cache Caffeine | Teste de carga; latência < 2 s no p95 sob pico simulado |
| RNF02 | Desempenho na busca de produtos | ADR-005 | `CatalogoService`, índice no banco | Consulta com filtro; latência < 1,5 s no p95 |
| RNF03 | Consistência de estoque sob concorrência | ADR-001 | `EstoqueService`, bloqueio otimista | Estoque nunca negativo após compras simultâneas automatizadas |
| RNF04 | Disponibilidade parcial (gateway indisponível) | ADR-004 | `PagamentoAdapter`, isolamento de falha | Simular gateway fora; catálogo e carrinho devem continuar operando |
| RNF05 | Resiliência a timeout de pagamento | ADR-004 | `PagamentoAdapter`, timeout configurado | Simular timeout; pedido deve ir para PENDENTE_PAGAMENTO |
| RNF06 | Segurança de dados | ADR-006 | Spring Security, JWT, cartão não armazenado (delegado ao gateway — ADR-004) | Revisão de código; teste de acesso não autorizado; ausência de dados sensíveis em logs |
| RNF07 | Integrabilidade — troca de gateway | ADR-004 | Interface `GatewayPagamento`, `PagamentoAdapter` | Trocar implementação do adaptador sem alterar `PedidoService` |

---

## Notas

> **Antes de entregar:** substituir os nomes de componentes pelos caminhos reais no repositório (ex.: `CatalogoService` → `src/main/java/.../catalogo/CatalogoService.java`) e confirmar que as ADRs referenciadas têm status `Aceito`.
