---
id: raiz.18
titulo: "Erros confirmados (raiz)"
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
tokens_estimados: 211
ficha: docs/modulos/README.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

## Erros confirmados (raiz)

| Situação | Retorno explicitamente tratado |
|---|---|
| Login sem `usuario`/`senha` | `400 {"mensagem":"Campos obrigatorios ausentes."}` |
| Login inválido | `401 {"mensagem":"Usuario ou senha invalidos."}` |
| Refresh ausente, revogado ou expirado | `401 {"mensagem":"Refresh token invalido ou expirado."}` |
| Conflito de agenda | `409` com `mensagem`, `conflitos` e `code="CONFLICT_HORARIOS"`. |

Não se deve transformar uma mensagem de erro textual em novo contrato; o código pode mudar.

