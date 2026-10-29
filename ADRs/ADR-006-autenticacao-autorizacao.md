# ADR-006 — Autenticação e autorização por perfil (Spring Security + RBAC)

- **Status:** Aceito
- **Data:** 2026-10-28

## Contexto

O RNF2 define que apenas usuários com perfil **lojista** ou **admin** podem cadastrar ou alterar preços de produtos. Outros perfis (cliente, entregador) têm acesso restrito a operações de leitura ou às próprias atribuições. O stakeholder Administrador exige que dados sensíveis de cartão nunca apareçam em relatórios ou logs. É necessário um mecanismo de autenticação e controle de acesso aplicado de forma centralizada e auditável.

## Decisão

Utilizar **Spring Security** com autenticação via **JWT (JSON Web Token)** e controle de acesso baseado em perfis (**RBAC — Role-Based Access Control**). Cada usuário recebe um ou mais perfis (ROLE_CLIENTE, ROLE_LOJISTA, ROLE_ENTREGADOR, ROLE_ADMIN). As rotas sensíveis (cadastro e alteração de preço de produto) são protegidas por anotações `@PreAuthorize` ou configuração de `SecurityFilterChain`, garantindo que apenas ROLE_LOJISTA e ROLE_ADMIN as acessem.

Dados de cartão não são armazenados no sistema — a responsabilidade é delegada ao gateway de pagamento (ver ADR-004).

## Alternativas consideradas

- **Sessões HTTP com estado (session-based):** descartado porque dificulta escalonamento horizontal futuro (sessões precisariam ser compartilhadas entre instâncias) e acoplam o cliente ao servidor de estado.
- **Sem controle de acesso centralizado (verificações manuais nos services):** descartado porque é propenso a falhas de implementação, não auditável e viola o RNF2 de forma sistemática.
- **OAuth2 com servidor de autorização externo:** descartado por complexidade de infraestrutura desnecessária nesta fase; pode ser adotado no Projeto Final se houver integração com provedores externos.

## Consequências

- **Positivas:** Controle de acesso centralizado e auditável; fácil extensão de perfis; JWT stateless facilita evolução para múltiplas instâncias; atende diretamente ao RNF2.
- **Negativas/trade-offs:** JWT requer estratégia de revogação (blacklist ou curto TTL + refresh token) para invalidação antes do vencimento. Requer cuidado com armazenamento seguro do segredo de assinatura (nunca no código-fonte).
