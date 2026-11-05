---
id: whatsapp.12
titulo: "Grupos (`/grupos`)"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 468
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Grupos (`/grupos`)

Leitura de grupos e integrantes, liberada por `perm:aba.suporte.whatsapp:Ler`. **O controller não cria nem edita**: a linha do grupo nasce do webhook e o título real é corrigido pela sincronização com o gateway.

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/grupos` | Query `numeroId?` | `200 List<GrupoWhatsApp>` (`id`, `numeroId`/`numeroDescricao`, `chatId`, `nome`, `ativo`); com `numeroId`, restringe aos grupos do número. `ListarAsync` sincroniza o título com o gateway quando o número é informado |
| `GET` | `/grupos/{id}/participantes` | Path `id` | `200 List<ParticipanteWhatsApp>` (`id`, `grupoId`, `nome`, `telefone`, `chatId`, `ativo`) dos **ativos**, ordenado por nome. Antes de ler, `ListarParticipantesAsync` chama `OpenWaClient.ObterGrupoAsync` (`GET /api/sessions/{sessaoId}/groups/{chatId}`) e faz **upsert por `WAT_ChatId`**: atualiza nome/telefone, reativa quem voltou e **desativa sem apagar** quem saiu. Lista vazia do gateway **não desativa** ninguém; falha do gateway é logada e a resposta é o cadastro local |
| `GET` | `/grupos/{id}/historico` | Path `id` | `200 HistoricoGrupo` com os atendimentos do grupo; `404` se o grupo não existe |

O endpoint de participantes é o que alimenta o modal "Integrantes" (o iniciar conversa aceita apenas JID privado e recusa `@g.us`). A sincronização é **tolerante**: o gateway é enriquecimento, o cadastro da Central é a resposta. O webhook cria apenas o participante que falou; sem essa sincronização o modal vinha vazio ou só com o solicitante.

