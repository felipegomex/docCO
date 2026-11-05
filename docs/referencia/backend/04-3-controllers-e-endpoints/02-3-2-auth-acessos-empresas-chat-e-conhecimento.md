---
id: 04-3-controllers-e-endpoints.02
titulo: "3.2 Auth, Acessos Empresas, Chat e Conhecimento"
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
tokens_estimados: 861
ficha: docs/modulos/acessos.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### 3.2 Auth, Acessos Empresas, Chat e Conhecimento

| Método e caminho | Corpo/parâmetros | Autorização | Comportamento |
|---|---|---|---|
| `POST /api/v1/auth/login` | `LoginRequest`: `usuario`, `senha`, `lembrarAcesso` | Público | Valida campos, autentica e grava cookie de refresh |
| `POST /api/v1/auth/refresh` | Sem parâmetro; frontend envia `{}` | Público | Rotaciona refresh pelo cookie `cc_refresh` |
| `POST /api/v1/auth/logout` | Sem corpo | JWT | Quando autorizado, revoga o refresh atual e apaga cookies; a chamada atual do frontend não envia Bearer |
| `GET /api/v1/auth/me` | Sem corpo | JWT | Obtém o operador pelo claim e devolve `UsuarioResponse` |
| `POST /api/v1/auth/...` — ver seção de Acessos abaixo | — | JWT | As **31** operações de `/api/v1/acessos` estão detalhadas em [`endpoints/raiz.md`](../../endpoints/raiz/INDEX.md) |
| `GET /api/chat/sso` | Sem corpo | JWT | Autentica a página `/chat` (iframe para o AnythingLLM); as rotas antigas (`/api/chat/agentes`, `/api/chat/perguntar`) foram removidas no `cb36ad3`; detalhes em [`endpoints/raiz.md`](../../endpoints/raiz/INDEX.md) |
| `/api/conhecimento/*` | Contratos de `ConhecimentoDtos.cs` | JWT, exceto callback OAuth; operações admin verificam papel | Navegação, revisão, PR, OAuth Gitea, auditoria e sincronização; detalhes em [`endpoints/raiz.md`](../../endpoints/raiz/INDEX.md) |

O controller de autenticação define [`LoginRequest`](../../../../backend/Central_BackEnd/Dtos/LoginRequest.cs), [`TokenResponse`](../../../../backend/Central_BackEnd/Dtos/TokenResponse.cs) e [`UsuarioResponse`](../../../../backend/Central_BackEnd/Dtos/UsuarioResponse.cs).

`AcessosController` é o **único** controller do módulo: rota `api/v{version:apiVersion}/acessos`, `[Authorize]` na classe **sem policy por papel** e 31 métodos `[Http*]`. As regras de negócio, os dois endpoints de revelação e o envelope `Paginado<T>` estão em [`regras-negocio.md`](../../regras-negocio/INDEX.md) e [`endpoints/raiz.md`](../../endpoints/raiz/INDEX.md).

O cookie `cc_refresh` é criado com `HttpOnly`, `SameSite=Strict`, `Secure=false` e `Path=/`. Com `lembrarAcesso=true`, recebe `MaxAge` de quatro horas e o controller cria o marcador `cc_lembrar=1`; sem persistência, o cookie de refresh é de sessão. A expiração da linha persistida em `tbrefreshtoken` vem de `Jwt:RefreshTokenHours`; `lembrarAcesso` altera a persistência do cookie, não esse campo. O campo `RefreshToken` é esvaziado no corpo antes da resposta.

