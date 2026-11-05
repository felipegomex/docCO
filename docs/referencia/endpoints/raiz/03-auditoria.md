---
id: raiz.03
titulo: "Auditoria"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 533
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

## Auditoria

Fonte: [AuditoriaController.cs](../../../../backend/Central_BackEnd/Controllers/AuditoriaController.cs) (`IAuditoriaLeituraService` em [IAuditoriaLeituraService.cs](../../../../backend/Central_BackEnd/Services/Auditoria/IAuditoriaLeituraService.cs), DTOs em [AuditoriaDtos.cs](../../../../backend/Central_BackEnd/Dtos/Auditoria/AuditoriaDtos.cs), envelope em [Paginado.cs](../../../../backend/Central_BackEnd/Dtos/Paginado.cs)).

**2 operações** em `api/v{version:apiVersion}/auditoria`, **somente leitura** (sem `POST`/`PUT`/`DELETE` — a trilha é append-only). É o **primeiro leitor** das tabelas de auditoria, até aqui só-escrita. Quem pode consultar é decidido no backend: **administrador** (role `Administrador`, claim `eh_admin = true` ou `perfil_id = A`) **OU gestor de função** (com ao menos uma função gerida em `tbgestor`); sem permissão o backend devolve **403** e a tela `/auditoria` avisa "sem acesso".

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/auditoria/acessos` | JWT (admin ou gestor; demais recebem `403`) | `validacao` | Query `usuario?`, `inicio?`, `fim?`, `page?`, `pageSize?` | `200 Paginado<AuditoriaAcessoItem>` (revelações de credencial, mais recentes primeiro; nunca traz senha). |
| `GET` | `/api/v1/auditoria/implantacao` | JWT (admin ou gestor; demais recebem `403`) | `validacao` | Query `entidade?`, `acao?`, `usuario?`, `inicio?`, `fim?`, `page?`, `pageSize?` | `200 Paginado<AuditoriaImplantacaoItem>` (eventos `CREATE`/`UPDATE`/`DELETE` com antes/depois em JSON, mais recentes primeiro). |

Paginação com o mesmo envelope `Paginado<T>` do Acessos (`items`, `totalCount`, `pageNumber`, `pageSize`, `totalPages`, `hasNext`, `hasPrev`; `PaginadoDefaults.Normalizar`: default 20, máximo 200).

