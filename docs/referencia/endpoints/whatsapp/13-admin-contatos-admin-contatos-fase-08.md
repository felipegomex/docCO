---
id: whatsapp.13
titulo: "Admin — contatos (`/admin/contatos`) — Fase 08"
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
tokens_estimados: 637
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Admin — contatos (`/admin/contatos`) — Fase 08

Leitura administrativa nova, separada do `GET /contatos` usado pelo overlay do atendimento (que **não** foi alterado — D03). **Todas as rotas sob `/admin` exigem administrador** (`IsInRole("Administrador")`, claim `eh_admin == "true"` ou `perfil_id == "A"`); sem o perfil, o controller responde **`403`**.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/admin/contatos` | **administrador** | Query `ContatoAdminFiltro`: `busca?` (nome, `chatId`, número ou telefone normalizado), `identificados?` (`true` = com empresa; `false` = pendentes; omitido = todos), `empresaId?`, `telefone?` (só os dígitos são comparados com `WAT_TelefoneNormalizado`), `pagina?` (1), `tamanho?` (20) | `200 Paginado<ContatoAdminDto>` — `id`, `nome`, `telefone`, `telefoneExibicao?`, `temFoto`, `fotoAtualizadaEm?`, `chatId`, `ehLid`, `empresaId`/`empresaNome`/`grupoEmpresaNome`, `tipoContato` + `tipoContatoNome`, `identificado`, `observacao`, `totalAtendimentos`, `ultimoAtendimentoEm`, `ultimoAtendimentoId` e `versao` (token do PATCH da Fase 01). Filtros aplicados **no servidor, antes de contar e paginar**; ordenado pelo último atendimento (sem atendimento vai ao fim). `totalAtendimentos`/`ultimoAtendimento*` saem de **subconsultas correlacionadas na mesma query paginada** — sem N+1 |
| `GET` | `/admin/contatos/{id}/historico` | **administrador** | Path `id` | `200 ContatoAdminHistoricoDto` (`contatoId`, `nome`, `chatId`, `ehLid`, `identificado`, `atendimentos[]` com status, motivo, fila, número, responsável, abertura, resolução e reaberturas), mais recente primeiro; **`404`** para contato inexistente |

**A escrita não foi duplicada.** Vínculo com a empresa, tipo de contato e observação continuam sendo gravados pelo **`PATCH /api/v1/whatsapp/contatos/{id}` da Fase 01**, com a `versao` devolvida por esta listagem e o lock otimista (`409` `CONFLICT_CONTATO_VERSAO` deixa o corpo com a ficha atual). Não existe endpoint novo de escrita nesta fase, e a ação continua auditada. O histórico usa a mesma regra do `HistoricoCliente`: todos os atendimentos das conversas do contato; o `@lid` não é normalizado nem comparado com telefone.

