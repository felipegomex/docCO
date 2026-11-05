---
id: whatsapp.09
titulo: "SLA (`/sla`)"
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
tokens_estimados: 409
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## SLA (`/sla`)

Leitura é de qualquer atendente; gravação e histórico são de administrador.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/sla` | atendente | — | `200 WhatsAppSlaConfigDto`: `metaPrimeiraRespostaMin`, `metaResolucaoMin`, **`percentualAviso`** (50..99, padrão 80 — Fase 05 do novo ciclo), `ativo`, `horarios`, `dataAlteracao` |
| `GET` | `/sla/historico` | **administrador** | — | `200 List<WhatsAppSlaHistoricoDto>` (campo, valor anterior, valor novo, usuário, data). `403` |
| `PUT` | `/sla/configuracao` | **administrador** | Body `SalvarSlaConfigRequest`: `metaPrimeiraRespostaMin?`, `metaResolucaoMin?`, **`percentualAviso?`** (50..99; fora da faixa → `400`), `ativo?` | `200 WhatsAppSlaConfigDto`; **cada campo alterado vira uma linha de histórico** (o campo do percentual é `percentualAviso`); `400` validação |
| `PUT` | `/sla/horarios` | **administrador** | Body `SalvarSlaHorariosRequest`: `horarios[]` (`id`, `diaSemana`, `inicio`, `fim`, `ativo`) | `200 List<WhatsAppSlaHorarioDto>`; **a lista enviada substitui a anterior**; `400` validação |

`diaSemana` segue o padrão **ISO**: `1` = segunda … `7` = domingo. **Não há tabela de feriados** — a decisão D6 do plano segue pendente; hoje o horário útil considera apenas o dia da semana.

