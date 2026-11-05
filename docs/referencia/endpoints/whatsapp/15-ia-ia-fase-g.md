---
id: whatsapp.15
titulo: "IA (`/ia`) — Fase G"
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
tokens_estimados: 324
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## IA (`/ia`) — Fase G

Configuração da IA do canal (admin) e assistente por atendimento (qualquer atendente com acesso ao canal).

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/ia/configuracao` | **administrador** | — | `200 IaConfigWhatsApp` (id, provedor, chaveApi mascarada, modelo, ativo, timeoutSeconds, usuarioAlteracao, dataAlteracao); `404` se não configurado |
| `PUT` | `/ia/configuracao` | **administrador** | Body `SalvarIaConfigRequest`: `provedor` ("Gemini"), `chaveApi`, `modelo`, `ativo`, `timeoutSeconds` | `200 IaConfigWhatsApp`; `400` validação |
| `GET` | `/ia/atendimentos/{id}/assistente-ia` | `perm:aba.suporte.whatsapp:Ler` | Path `id` (atendimento) | `200 AssistenteIaResumoWhatsApp` (atendimentoId, resumo, categoria, prioridade, sentimento, intencao, respostasSugeridas[], artigosSugeridos[], procedimentosSugeridos[], proximosPassos[]); `404` se atendimento não existe ou IA não configurada |

