---
id: mod.kanban
titulo: "Kanban Geral"
tipo: modulo
area: implantacao
sistema: central
tags: [kanban, tarefa, projeto, funcao, bloqueio, gargalo, implantacao]
dono: implantacao
status: ativo
revisado_em: "2026-10-29"
fonte: [backend/Central_BackEnd/Controllers/Implantacao, backend/Central_BackEnd/Services/Implantacao, backend/Central_BackEnd/Models/Implantacao, frontend/src/app/features/implantacao/pages/kanban]
testes: [backend/Central_BackEnd.Tests/KanbanFuncaoTests.cs, frontend/src/app/features/implantacao/pages/kanban/kanban.component.spec.ts]
tokens_estimados: 1900
---

# Kanban Geral

## 1. Quando usar este card

Quadro por função, mover card, catálogo de colunas, visão do setor, bloqueios e
gargalos, etapas da função, permissão por função e deep links `?funcaoId=`.

## 2. Propósito e escopo

Quadro único com **eixo na função** (`tbfuncao`): tarefas e projetos no mesmo
board, colunas compartilhadas com visão por setor, bloqueio como atributo e
gargalos. Não cobre o Kanban de atendimento (`whatsapp.md`) nem o CRUD completo
de tarefa/projeto (os controllers ficam em `docs/referencia/backend/`).

## 3. Rotas do frontend

| URL | Componente | Fonte |
|---|---|---|
| `/kanban` | `KanbanComponent` (modo geral) | `frontend/src/app/features.routes.ts` |
| `/kanban/gargalos` | `GargalosComponent` | idem |
| `/kanban/configurar` | `ConfigColunasSetorComponent` | idem |
| `/kanban/admin/colunas` | `AdminColunasComponent` | idem |
| `/implantacao/kanban` | redirect para `/kanban` | `frontend/src/app/features/implantacao/implantacao.routes.ts` |

## 4. Endpoints do backend

| Rotas | Autorização | Fonte |
|---|---|---|
| `GET /implantacao/kanban/board`, `POST /cards/{tipo}/{id}/coluna` | `[Authorize]` + regra de função | `backend/Central_BackEnd/Controllers/Implantacao/KanbanController.cs` |
| `GET /implantacao/funcoes` | `[Authorize]` | `backend/Central_BackEnd/Controllers/Implantacao/FuncoesController.cs` |
| CRUD `/implantacao/etapas-funcao` (6 operações) | escrita `Administrador` | `backend/Central_BackEnd/Controllers/Implantacao/EtapasFuncaoController.cs` |
| `GET/POST /implantacao/bloqueios` | `[Authorize]` | `backend/Central_BackEnd/Controllers/Implantacao/BloqueiosController.cs` |
| `GET /implantacao/gargalos` | `[Authorize]` | idem |
| `GET/POST/PUT /implantacao/colunas-kanban` (6 operações) | catálogo `Administrador`; visão gestor | `backend/Central_BackEnd/Controllers/Implantacao/ColunasKanbanController.cs` |

## 5. Tabelas e entidades

| Classe → tabela | Papel | Fonte |
|---|---|---|
| `Funcao` → `tbfuncao` | Eixo do quadro | `backend/Central_BackEnd/Models/Funcao.cs` |
| `ColunaKanban` → `tbcolunakanban` | Catálogo compartilhado | `backend/Central_BackEnd/Models/Implantacao/ColunaKanban.cs` |
| `ColunaFuncao` → `tbcolunafuncao` | Visão do setor (visível + máscara) | `backend/Central_BackEnd/Models/Implantacao/ColunaFuncao.cs` |
| `Tarefa` → `tbtarefa`; `Projeto` → `tbprojeto` | Cards | `backend/Central_BackEnd/Models/Implantacao/Tarefa.cs` |
| `EtapaFuncao` → `tbfuncaoetapa` | Etapas fixas | `backend/Central_BackEnd/Models/Implantacao/EtapaFuncao.cs` |
| `Bloqueio` → `tbbloqueio`; `Operador` → `tboperador` | Bloqueio e função do operador | `backend/Central_BackEnd/Models/Implantacao/Bloqueio.cs` |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `FuncaoAuthorizationService` | Permissão por função | `backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs` |
| backend | `KanbanBoardService` | Board e movimento | `backend/Central_BackEnd/Services/Implantacao/KanbanBoardService.cs` |
| backend | `ColunaKanbanService` | Catálogo e visão do setor | `backend/Central_BackEnd/Services/Implantacao/ColunaKanbanService.cs` |
| backend | `BloqueioService` | Bloqueios e gargalos | `backend/Central_BackEnd/Services/Implantacao/BloqueioService.cs` |
| backend | `RegrasPrazo` | Decide atraso | `backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs` |
| frontend | `KanbanComponent` | Board, pills, permissão espelho | `frontend/src/app/features/implantacao/pages/kanban/kanban.component.ts` |
| frontend | `AdminColunasComponent`, `ConfigColunasSetorComponent` | Catálogo e visão | `frontend/src/app/features/implantacao/pages/kanban/admin-colunas/admin-colunas.component.ts` |

## 7. Regras de negócio

- A função é o eixo; o front sempre envia `funcaoId` no board.
- Catálogo de colunas é compartilhado; o setor controla só `CLF_Visivel` e
  `CLF_NomeAlternativo` (`ColunaKanbanService#SalvarVisaoAsync`).
- Coluna não é excluída: desativação global é o caminho (`AtualizarAsync`).
- Mover card não abre/fecha bloqueio; bloqueio tem motivo e no máximo 1 aberto
  por origem (`BloqueioService#AbrirAsync`).
- `GET /implantacao/funcoes` é somente leitura; função é mantida no operador.
- O backend decide `atrasado`/`diasAtraso`/`dataConclusao` (`RegrasPrazo`),
  inclusive no `TarefaDetalhe`.

## 8. Autorização

- Leitura valida `PodeVerFuncao`; escrita valida `PodeEditar`.
- Ver/editar a própria função ou as geridas; `GESTOR_MASTER`/admin vê qualquer
  função; card sem função só o Master; customizar é Master ou gestor.
- Catálogo de colunas/etapas: `[Authorize(Roles = "Administrador")]`.
- Visão do setor: gestor da função ou administrador; demais 403.
- O espelho no frontend (`KanbanComponent.podeEditarCard`) é UX; a fonte é o
  backend.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “mover/reordenar card” | `backend/Central_BackEnd/Services/Implantacao/KanbanBoardService.cs#MoverCardAsync` | `frontend/src/app/features/implantacao/pages/kanban/kanban.component.ts#KanbanComponent` | `backend/Central_BackEnd.Tests/KanbanFuncaoTests.cs` |
| “criar/editar coluna” | `backend/Central_BackEnd/Services/Implantacao/ColunaKanbanService.cs#CriarAsync` | `frontend/src/app/features/implantacao/pages/kanban/admin-colunas/admin-colunas.component.ts#AdminColunasComponent` | `backend/Central_BackEnd.Tests/KanbanFuncaoTests.cs` |
| “visão/máscara do setor” | `backend/Central_BackEnd/Services/Implantacao/ColunaKanbanService.cs#SalvarVisaoAsync` | `frontend/src/app/features/implantacao/pages/kanban/config-colunas-setor/config-colunas-setor.component.ts#ConfigColunasSetorComponent` | `backend/Central_BackEnd.Tests/KanbanFuncaoTests.cs` |
| “bloqueio” | `backend/Central_BackEnd/Services/Implantacao/BloqueioService.cs#AbrirAsync` | `frontend/src/app/features/implantacao/pages/kanban/kanban.component.ts#KanbanComponent` | `backend/Central_BackEnd.Tests/KanbanFuncaoTests.cs` |
| “permissão por função” | `backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs#FuncaoAuthorizationService` | `frontend/src/app/features/implantacao/pages/kanban/kanban.component.ts#KanbanComponent` | `backend/Central_BackEnd.Tests/GestorFuncaoTests.cs` |
| “atraso/prazo” | `backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs#RegrasPrazo` | — | `backend/Central_BackEnd.Tests/RegrasPrazoTests.cs` |

## 10. Contratos que não podem quebrar

- `limite` padrão 1000, sinalizado por `truncado`/`limiteAplicado`.
- Coluna de destino precisa estar visível no quadro do setor; senão 400.
- Desativar coluna com cards é recusado; bloqueio aberto é único por origem.

## 11. Armadilhas conhecidas

- `dotnet ef database update` não deve ser usado: `Up` vazio de propósito e DDL
  real em `Migrations/Sql/*_Idempotente.sql`.
- Cor de card/pill vem de `CLASSIFICACAO`, não do id.

## 12. Testes

- `KanbanFuncaoTests`, `GestorFuncaoTests`, `RegrasPrazoTests`,
  `PerfilOperadorTests` e `TarefaDataConclusaoTests`;
  `kanban.component.spec.ts`.
- QA: `docs/referencia/qa/06-3-1-kanban-geral.md` (KG-01 a KG-03).

## 13. Orçamento de leitura

Abra esta ficha; depois só o arquivo da seção 9 e o teste. Limite: 1 ficha,
4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- Mudou a matriz de permissão? Atualizar backend **e**
  `KanbanComponent.podeEditarCard` juntos.
