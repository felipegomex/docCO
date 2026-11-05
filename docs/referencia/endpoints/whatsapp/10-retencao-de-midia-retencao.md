---
id: whatsapp.10
titulo: "Retenção de mídia (`/retencao`)"
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
tokens_estimados: 347
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Retenção de mídia (`/retencao`)

Tudo é de administrador: a regra decide quanto o banco guarda de mídia e a execução manual mexe no histórico.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/retencao` | **administrador** | — | `200 WhatsAppRetencaoStatusDto`: regra vigente, última execução, bytes de mídia/payload, mensagens com mídia e total |
| `PUT` | `/retencao/configuracao` | **administrador** | Body `SalvarRetencaoConfigRequest`: `diasRetencao?`, `ativo?` | `200 WhatsAppRetencaoConfigDto`; `400` validação |
| `POST` | `/retencao/executar` | **administrador** | — | `200 WhatsAppRetencaoExecucaoDto` (origem `Manual`, contagens de mensagens/payloads limpos, bytes liberados, erro) |

A limpeza atua **somente no base64** (`WAT_MidiaBase64` e `WAT_Payload`): metadados (`WAT_MidiaMime`, `WAT_MidiaNome`) permanecem, **nenhum atendimento aberto é tocado** e o conteúdo no gateway fica fora do escopo (decisão D1). A execução manual chama o mesmo serviço do job agendado — muda apenas a origem registrada.

