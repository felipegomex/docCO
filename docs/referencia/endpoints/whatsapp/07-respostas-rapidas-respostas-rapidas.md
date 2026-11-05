---
id: whatsapp.07
titulo: "Respostas rápidas (`/respostas-rapidas`)"
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
tokens_estimados: 404
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Respostas rápidas (`/respostas-rapidas`)

Listar e obter são de qualquer atendente — a resposta rápida é texto de trabalho. **Criar, editar e inativar são de administrador**; a validação é do backend (`403` para os demais). Nenhuma escrita envia mensagem: a resposta só insere texto no compositor.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/respostas-rapidas` | atendente | Query `somenteAtivos` (padrão `true`), `busca?` | `200 List<RespostaRapidaWhatsApp>` |
| `GET` | `/respostas-rapidas/{id}` | atendente | Path | `200 RespostaRapidaWhatsApp`; `404` |
| `POST` | `/respostas-rapidas` | **administrador** | Body `CriarRespostaRapidaRequest`: `atalho?` (1–40), `categoria?` (≤ 50), `titulo?` (≤ 120), `corpo?` (≤ 4000) | `201 RespostaRapidaWhatsApp`; `400` validação; **`409` atalho repetido** |
| `PATCH` | `/respostas-rapidas/{id}` | **administrador** | Body `AtualizarRespostaRapidaRequest`: mesmos campos + `ativo?` | `200 RespostaRapidaWhatsApp`; `404`; `409` atalho repetido |
| `DELETE` | `/respostas-rapidas/{id}` | **administrador** | — | `204`. **Inativa** (`WAT_Ativo = 0`), não apaga |

O `corpo` aceita as variáveis `{nome}`, `{protocolo}` e `{numero}`; a substituição acontece **na tela**, antes da revisão do atendente.

