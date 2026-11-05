---
id: raiz.15
titulo: "Chat de Conhecimento"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Controllers/ChatSsoController.cs
testes: []
tokens_estimados: 420
ficha: docs/modulos/chat-conhecimento.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Chat e Conhecimento](../../../modulos/chat-conhecimento.md).

## Chat de Conhecimento

Fonte: [ChatSsoController.cs](../../../../backend/Central_BackEnd/Controllers/ChatSsoController.cs).

O chat é servido pelo AnythingLLM embutido em iframe; a Central só realiza o SSO.
Os endpoints `/api/chat/agentes`, `/api/chat/perguntar` e `/api/chat/provedor`
foram removidos junto com `RagProxyController`/`ChatPerguntarService` (commit
`cb36ad3`).

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/api/chat/sso` | JWT | Sem body | `200` com `{ loginPath, url, token }`; `401` sem identidade; `404` operador sem usuário no AnythingLLM; `502` configuração ausente ou falha do AnythingLLM; `504` timeout. |

É uma exceção sem versão (`/api/chat`), por isso o front usa `apiRootUrl`
(host puro). O fluxo: lê o `sub` do JWT, procura o `username` correspondente em
`GET {AnythingLLM:BaseUrl}/admin/users`, emite token em
`GET /users/{id}/issue-auth-token` e monta `url` com `AnythingLLM:InstanceUrl`
(ou `BaseUrl` sem `/api/v1`). O token é temporário e não vai para log.

Os endpoints do módulo Conhecimento continuam em
[ConhecimentoController.cs](../../../../backend/Central_BackEnd/Controllers/ConhecimentoController.cs)
e estão descritos na [ficha do módulo](../../../modulos/chat-conhecimento.md).
