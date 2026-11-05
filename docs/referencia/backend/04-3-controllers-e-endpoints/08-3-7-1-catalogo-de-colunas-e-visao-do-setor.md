---
id: 04-3-controllers-e-endpoints.08
titulo: "3.7.1 Catálogo de colunas e visão do setor"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 755
ficha: docs/modulos/database-explorer.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 3.7.1 Catálogo de colunas e visão do setor

O **catálogo** (`tbcolunatecanban`, `CLK_FuncaoId` nulo) é compartilhado: as mesmas colunas para todos os setores, porque a análise comparativa entre funções exige o mesmo conjunto. O que o setor controla é a sua **visão** (`tbcolunafuncao`): `CLF_Visivel` oculta/ativa a coluna no quadro do setor e `CLF_NomeAlternativo` é a **máscara de nome** (como a coluna aparece naquele setor, sem renomear a do catálogo).

| Método e caminho | Comportamento |
|---|---|
| `GET /implantacao/colunas-kanban` | `todas=true` → catálogo inteiro (`ListarTodasAsync`); `funcaoId=N` → **visão do setor** (`ListarQuadroAsync`: catálogo ativo filtrado pela visibilidade, com a máscara aplicada no campo `nome`); sem parâmetros → catálogo ativo (`ListarAsync`) |
| `GET /implantacao/colunas-kanban/visao/{funcaoId}` | Visão completa do setor: **todas** as colunas do catálogo (inclusive as inativas) com o que o setor definiu — `visivel` e `nomeAlternativo` — e o nome do catálogo em `nomeCatalogo`. `403` para quem não é gestor da função nem administrador |
| `PUT /implantacao/colunas-kanban/visao/{funcaoId}` | Grava a visão: cria/atualiza as linhas de `tbcolunafuncao` do setor. Coluna fora do catálogo → `400 "Coluna fora do catálogo: …"`; função inexistente ou inativa → `400 "Função informada não existe."`; `funcaoId` do corpo divergente da rota → `400 "FuncaoId do corpo diverge da rota."` |
| `POST /implantacao/colunas-kanban` | Cria coluna **do catálogo** (`FuncaoId = null`, ignorado o `funcaoId` do corpo) — exige `Roles = "Administrador"` |
| `PUT /implantacao/colunas-kanban/{id}` | Edita nome/ordem/cor/limite WIP/status/agenda e ativa/desativa. Desativar coluna **com cards** é recusado (`400 "Não é possível desativar: existem cards nesta coluna."`) |
| `POST /implantacao/colunas-kanban/reordenar` | Reordena o **catálogo inteiro** (`idsEmOrdem`): uma ordem só, para todos os setores. Sequência vazia, id repetido ou id inexistente → `400 { mensagem }` |

`ColunaBoard` (o quadro) devolve `nome` = nome efetivo (com a máscara do setor) e `nomeCatalogo` = nome do catálogo; `funcaoId`/`funcaoNome` da coluna vêm **sempre nulos** — a coluna é do catálogo, não da função.

> ⚠️ `GET /api/v1/admin/dashboard` **não tem tela**. O `AdminDashboardComponent` e o `AdminDashboardService` do **frontend** foram removidos em 2026-10-20 (Fase 0 do plano `qualidade-logica`), mas o `AdminDashboardController` e o `IAdminDashboardService` **continuam existindo e registrados no DI** (`Program.cs`). O par é contrato de API, não funcionalidade de tela — nenhum item de menu ou rota do frontend o consome.

