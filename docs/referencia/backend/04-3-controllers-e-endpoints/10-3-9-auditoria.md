---
id: 04-3-controllers-e-endpoints.10
titulo: "3.9 Auditoria"
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
tokens_estimados: 287
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

### 3.9 Auditoria

`AuditoriaController` (`api/v{version}/auditoria`, `[Authorize]`, limite `validacao`) expõe **somente leitura** das duas trilhas via `IAuditoriaLeituraService` (registrado scoped em `Program.cs`); a permissão (administrador OU gestor de função, senão `403`) é decidida no backend. Detalhe das rotas em [05-ENDPOINTS.md](../../endpoints.md).

| Método e caminho | Corpo/parâmetros | Resultado principal |
|---|---|---|
| `GET /api/v1/auditoria/acessos` | `usuario?`, `inicio?`, `fim?`, `page?`, `pageSize?` | `Paginado<AuditoriaAcessoItem>` (revelações, mais recentes primeiro; nunca traz senha) |
| `GET /api/v1/auditoria/implantacao` | `entidade?`, `acao?`, `usuario?`, `inicio?`, `fim?`, `page?`, `pageSize?` | `Paginado<AuditoriaImplantacaoItem>` (`CREATE`/`UPDATE`/`DELETE` com antes/depois em JSON) |

