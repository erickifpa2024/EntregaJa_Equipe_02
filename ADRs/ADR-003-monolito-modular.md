# ADR-003 — Monólito modular em camadas como arquitetura-alvo da E2

- **Status:** Aceito
- **Data:** 2026-10-28

## Contexto

O domínio do EntregaJá é suficientemente complexo para exigir separação de responsabilidades, mas não há justificativa operacional para implantação distribuída nesta fase. O guia da E2 orienta explicitamente a não desenhar microsserviços separados, salvo decisão explícita da equipe.

## Decisão

Aplicação única Spring Boot 3 / Java 17, organizada em cinco camadas e seis módulos de negócio internos:

**Camadas:**
| Camada | Responsabilidade |
|--------|-----------------|
| Apresentação | Receber requisições e validar entradas (Controllers/DTOs) |
| Aplicação | Orquestrar casos de uso e transações (Services) |
| Domínio | Aplicar regras de negócio e controlar estados (entidades, value objects) |
| Persistência | Salvar e consultar informações (Repositories, JPA) |
| Integração | Comunicar com sistemas externos e publicar eventos (Adapters) |

**Módulos de negócio:** Catálogo, Pedido/Carrinho, Estoque, Pagamento, Entrega, Segurança.

Fronteiras internas mantidas por pacotes e interfaces, não por serviços separados. A camada de domínio não depende diretamente de infraestrutura.

## Alternativas consideradas

- **Aplicação sem separação em camadas:** descartado porque dificulta testes unitários, manutenção e a futura evolução para serviços no Projeto Final.
- **Microsserviços desde o início:** descartado porque multiplica complexidade operacional (deploy, rede, observabilidade distribuída) sem que os requisitos da E2 o exijam.

## Consequências

- **Positivas:** Implantação única simplifica o ambiente; testes facilitados com fronteiras bem definidas por camada.
- **Negativas/trade-offs:** Risco de degradação das fronteiras internas com o tempo — mitigado pela disciplina de acessar módulos apenas via suas interfaces definidas e não cruzar camadas diretamente.
