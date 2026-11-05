---
id: raiz.01
titulo: "Autenticação"
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
tokens_estimados: 370
ficha: docs/modulos/README.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

## Autenticação

Fonte: [AuthController.cs](../../../../backend/Central_BackEnd/Controllers/AuthController.cs).

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `POST` | `/api/v1/auth/login` | Público | `login` | Body `LoginRequest`: `usuario`, `senha`, `lembrarAcesso` | `200 TokenResponse`; o campo `refreshToken` permanece no JSON como `refreshToken: ""`. `400` para campos ausentes e `401` para credenciais inválidas. |
| `POST` | `/api/v1/auth/refresh` | Público | — | Sem body necessário; o frontend envia `{}` | `200 TokenResponse`; renova e rotaciona o cookie, mantendo no JSON `refreshToken: ""`. `401` se o cookie estiver ausente, revogado ou expirado. |
| `POST` | `/api/v1/auth/logout` | JWT | — | Sem body | `204`; revoga o refresh enviado e limpa os cookies. |
| `GET` | `/api/v1/auth/me` | JWT | — | Sem body | `200 UsuarioResponse`; `401` para token sem operador e `404` para usuário não encontrado. |

O login aceita `Usuario` como `OperadorId` ou `Email`. A resposta inclui `AccessToken`, `ExpiraEmSegundos` e `User`; o refresh bruto não deve ser tratado como campo de resposta utilizável.

