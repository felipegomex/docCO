---
id: mod.agenda
titulo: "Agenda"
tipo: modulo
area: agenda
sistema: central
tags: [agenda, evento, compromisso, recorrencia, serie, conflito, visibilidade, operador]
dono: operacao
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/AgendaController.cs
  - backend/Central_BackEnd/Services/Implantacao/AgendaService.cs
  - backend/Central_BackEnd/Models/Implantacao/AgendaItem.cs
  - frontend/src/app/features/agenda
testes:
  - backend/Central_BackEnd.Tests/AgendaRecorrenciaTests.cs
  - backend/Central_BackEnd.Tests/AgendaVisibilidadeTests.cs
  - frontend/src/app/features/agenda/agenda.component.spec.ts
tokens_estimados: 1800
---

# Agenda

## 1. Quando usar este card

Use para: criar/editar/excluir compromisso, visão dia/semana/operacional, evento
recorrente e série, conflito de horário (409), visibilidade, responsável, função,
tipo de evento e prioridade.

## 2. Propósito e escopo

Agenda de compromissos por operador/função, com recorrência, série e detecção
de conflito. Não cobre o prazo/atraso do Kanban (`RegrasPrazo`) nem o SLA do
WhatsApp.

## 3. Rotas do frontend

| URL | Componente | Observação | Fonte |
|---|---|---|---|
| `/agenda` | `AgendaComponent` | Visões dia/semana/operacional | `frontend/src/app/features.routes.ts` |
| modal | `AgendaEventoModalComponent` | Criar/editar evento | `frontend/src/app/features/agenda/agenda-evento-modal.component.ts` |

A rota `/agenda` exige `aba.inicio.agenda`
(`frontend/src/app/features.routes.ts`).

## 4. Endpoints do backend

| Método | Rota | Autorização | Observação | Fonte |
|---|---|---|---|---|
| GET | `/api/v{version}/agenda/eventos` | `[Authorize]` | Lista por período, responsável e função | `backend/Central_BackEnd/Controllers/AgendaController.cs` |
| GET | `/agenda/eventos/{id}` | idem | Detalhe | idem |
| POST | `/agenda/eventos` | idem | Cria | idem |
| POST | `/agenda/eventos/lote` | idem | Cria série | idem |
| PUT | `/agenda/eventos/{id}` | idem | Atualiza | idem |
| DELETE | `/agenda/eventos/{id}` | idem | Exclui | idem |
| PATCH | `/agenda/eventos/{id}/mover` | idem | Move horário | idem |
| GET | `/agenda/tipos`, `/funcoes`, `/operadores` | idem | Lookups | idem |

## 5. Tabelas e entidades

| Classe | Tabela | Papel | Fonte |
|---|---|---|---|
| `AgendaItem` | `tbagenda` | Evento, recorrência e série | `backend/Central_BackEnd/Models/Implantacao/AgendaItem.cs` |
| `AgendaParticipante` | `tbagendaparticipante` | Participantes do evento | `backend/Central_BackEnd/Models/Implantacao/AgendaParticipante.cs` |
| `TipoEvento` | `tbtipoevento` | Catálogo de tipos | `backend/Central_BackEnd/Models/Implantacao/TipoEvento.cs` |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `AgendaService` | CRUD, série, conflito e lookups | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs` |
| frontend | `AgendaService` | HTTP da agenda | `frontend/src/app/features/agenda/services/agenda.service.ts` |
| frontend | `AgendaComponent` | Grade e filtros | `frontend/src/app/features/agenda/agenda.component.ts` |
| frontend | `AgendaEventoModalComponent` | Formulário do evento | `frontend/src/app/features/agenda/agenda-evento-modal.component.ts` |

## 7. Regras de negócio

- Evento tem `Tipo` (`AgendaTipo`), `Visibilidade`, `Prioridade` e recorrência
  (`PadraoRecorrencia`) — `AgendaItem`.
- Série é identificada por `SerieId`; o lote grava o mesmo `SerieId` em todas as
  instâncias (`AgendaRecorrenciaTests`).
- Edição com fim de repetição recria o “buraco” da série e estreitar remove os
  irmãos após o novo fim (`AgendaRecorrenciaTests`).
- Conflito de horário do responsável bloqueia a criação com `409`
  (`AgendaService#ObterConflitosAsync`).
- Fim de repetição anterior ao início é rejeitado.
- Edição de conteúdo não se propaga aos irmãos; só o recorte da série muda.
- Evento de projeto pode referenciar `ProjetoId`.
- **Visibilidade**: `Publico` aparece para todos; `Privado` só para o
  responsável, participantes, gestor da função do responsável ou admin
  (`AgendaService#FiltroVisibilidade`).

## 8. Autorização

- `[Authorize]` em todos os endpoints; `isAdmin` é derivado de
  `User.IsInRole("Administrador")` ou claim `perfil` em `AgendaController#IsAdmin`.
- O frontend exige `aba.inicio.agenda`; o backend revalida no serviço.
- A listagem e o detalhe aplicam `FiltroVisibilidade`: privado não é devolvido a
  quem não é dono, participante, gestor da função ou admin.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “criar evento” | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs#CriarEventoAsync` | `frontend/src/app/features/agenda/agenda-evento-modal.component.ts#AgendaEventoModalComponent` | `backend/Central_BackEnd.Tests/AgendaRecorrenciaTests.cs` |
| “criar série/lote” | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs#CriarEventosLoteAsync` | `frontend/src/app/features/agenda/agenda-evento-modal.component.ts#AgendaEventoModalComponent` | `backend/Central_BackEnd.Tests/AgendaRecorrenciaTests.cs` |
| “editar evento/série” | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs#AtualizarEventoAsync` | `frontend/src/app/features/agenda/agenda-evento-modal.component.ts#AgendaEventoModalComponent` | `backend/Central_BackEnd.Tests/AgendaRecorrenciaTests.cs` |
| “mover horário” | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs#MoverEventoAsync` | `frontend/src/app/features/agenda/agenda.component.ts#AgendaComponent` | `backend/Central_BackEnd.Tests/AgendaRecorrenciaTests.cs` |
| “conflito de horário” | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs#ObterConflitosAsync` | `frontend/src/app/features/agenda/agenda-evento-modal.component.ts#AgendaEventoModalComponent` | `backend/Central_BackEnd.Tests/AgendaRecorrenciaTests.cs` |
| “visão dia/semana/operacional” | — | `frontend/src/app/features/agenda/agenda.component.ts#AgendaComponent` | `frontend/src/app/features/agenda/agenda.component.spec.ts` |
| “tipos de evento” | `backend/Central_BackEnd/Services/Implantacao/AgendaService.cs#ListarTiposAsync` | `frontend/src/app/features/agenda/agenda.component.ts#AgendaComponent` | — (sem teste dedicado) |

## 10. Contratos que não podem quebrar

- `SerieId` é o vínculo da recorrência; não gerar um novo por instância.
- Conflito de horário devolve `409`; não transformar em `400`.
- Exclusão/edição respeita o escopo da série (`idExcluir` nos conflitos).
- O `OperadorId` do evento é o responsável; não trocar por função.
- `ProjetoId` é opcional; não quebrar evento avulso.

## 11. Armadilhas conhecidas

- Série legada sem `SerieId` é adotada na primeira edição — comportamento testado.
- Alterar o fim da repetição pode apagar irmãos; é intencional.
- Fuso/horário: a API formata data; não comparar `DateTime` sem normalizar.
- Evento privado não aparece para outro operador (comportamento intencional).

## 12. Testes

- Backend: `AgendaRecorrenciaTests` (série, edição, conflito `409`, fim anterior
  ao início, geração semanal) e `AgendaVisibilidadeTests` (privado por dono,
  gestor, admin e detalhe).
- Frontend: `agenda.component.spec.ts` (criação e estado inicial).
- QA: `docs/referencia/qa/10-5-empresa-agenda.md`.

## 13. Orçamento de leitura

- Abra primeiro esta ficha.
- Abra depois só o serviço/controller da seção 9 e o teste correspondente.
- Limite: 1 ficha, 4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- (nenhuma — visibilidade aplicada e coberta por testes).
