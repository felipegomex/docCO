---
id: raiz.17
titulo: "Exemplos mínimos confirmados (raiz)"
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
tokens_estimados: 322
ficha: docs/modulos/README.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

## Exemplos mínimos confirmados (raiz)

Os placeholders abaixo não são segredos nem valores de configuração.

### Login e cookie

```http
POST /api/v1/auth/login
Content-Type: application/json

{"usuario":"operador.exemplo","senha":"<valor-nao-real>","lembrarAcesso":true}
```

O token de acesso é devolvido no corpo e deve ficar em memória no cliente. O refresh token não deve ser copiado para armazenamento local; o backend o envia em `cc_refresh` HttpOnly.

```http
Authorization: Bearer <token-em-memória>
```

```http
POST /api/v1/auth/refresh
Cookie: cc_refresh=<cookie-http-only>
```

### Agenda

```json
{
  "titulo": "Reunião de acompanhamento",
  "descricao": "Evento de exemplo",
  "local": "Sala interna",
  "dataInicio": "2026-09-24T13:00:00",
  "dataFim": "2026-09-24T14:00:00",
  "diaInteiro": false,
  "tipoId": null,
  "responsavelId": "operador.exemplo",
  "projetoId": null,
  "participantesIds": [],
  "prioridade": 0,
  "slaMinutos": 60
}
```

