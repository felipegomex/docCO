---
id: 04-3-controllers-e-endpoints.01
titulo: "3.1 Convenções"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 237
ficha: docs/modulos/README.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 3.1 Convenções

- Controllers de domínio usam `api/v{version:apiVersion}` e `[ApiVersion("1.0")]`.
- A versão pode vir do segmento de URL ou de `X-Api-Version`; a versão padrão configurada é 1.0, mas todos os controllers versionados usam uma rota que contém o segmento versionado.
- Chat (`/api/chat`) e Conhecimento (`/api/conhecimento`) são exceções sem versão; `/api/rag-proxy` não é mais a rota ativa.
- “JWT” significa `[Authorize]`; “Admin” significa `[Authorize(Roles = "Administrador")]`; “Público” significa ausência de `[Authorize]`.
- Os corpos abaixo são resumos; os campos completos estão nos DTOs citados.

