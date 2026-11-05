---
id: whatsapp.08
titulo: "Configuração e monitor"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 211
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Configuração e monitor

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/configuracao` | **administrador** | — | `200 WhatsAppConfiguracaoDto`: checklist de presença (tabelas esperadas), números, fila e versão do gateway. `403` |
| `GET` | `/configuracao/finalizacao-automatica` | **administrador** | — | `200 FinalizacaoAutomaticaConfig` (`{ minutos }`); lê `atendimento.finalizacaoAutomaticaMinutos` em `IWhatsAppParametroService` e cai no padrão **15** quando ausente. `403` |
| `PUT` | `/configuracao/finalizacao-automatica` | **administrador** | Body `FinalizacaoAutomaticaConfig`: `minutos` (`0..1440`) | `200` com o mesmo DTO; grava o parâmetro com o operador do token como autor. `400 { mensagem }` fora da faixa (0 desliga a rodada); `403` sem perfil |
| `GET` | `/monitor` | atendente | — | `200 WhatsAppMonitorDto`: saúde por número e estado da fila |

**Nenhuma resposta carrega valor de segredo**: chave da API e segredo do webhook aparecem, no máximo, como "configurado / não configurado".

