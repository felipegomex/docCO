---
id: whatsapp.14
titulo: "Auditoria (`/auditoria`) — Fase F"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 213
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Auditoria (`/auditoria`) — Fase F

Leitura paginada do log genérico de ações no canal (`tbwhatsapp_auditoria`). **Só administrador**.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/auditoria` | `perm:aba.suporte.whatsapp:Ler` + **administrador** | Query `AuditoriaFiltro`: `usuarioId?`, `acao?`, `objeto?`, `periodoDe?`, `periodoAte?`, `pagina` (1), `tamanho` (20) | `200 Paginado<AuditoriaWhatsApp>` (id, usuarioId, usuarioNome, dataHora, acao, objeto, objetoId, alteracao) |

