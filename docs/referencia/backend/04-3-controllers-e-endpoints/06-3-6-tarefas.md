---
id: 04-3-controllers-e-endpoints.06
titulo: "3.6 Tarefas"
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
tokens_estimados: 1325
ficha: docs/modulos/kanban.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 3.6 Tarefas

Todos os endpoints usam JWT e `validacao`.

| Método e caminho | Corpo/parâmetros | Autorização/efeito |
|---|---|---|
| `GET /api/v1/implantacao/tarefas` | Filtros de Projeto, responsável, status, datas, perfil, função e arquivamento | Lista até 500; arquivadas excluídas por padrão; `403` quando `funcaoId` é setor proibido para o operador |
| `GET /api/v1/implantacao/tarefas/{id}` | Sem corpo | Detalhe completo |
| `POST /api/v1/implantacao/tarefas` | `TarefaCriarRequest` | Cria tarefa, vínculos e auditoria; `projetoEtapaId` sem `projetoId` é rejeitado com mensagem explícita |
| `PUT /api/v1/implantacao/tarefas/{id}` | `TarefaAtualizarRequest` | Atualiza os campos enviados, sincroniza card/coluna e audita; não é um PATCH de campos parciais; não troca `projetoId` e valida/zera `projetoEtapaId` |
| `PATCH /api/v1/implantacao/tarefas/{id}/coluna` | `TarefaMudarColunaRequest` | Move, aplica WIP e sincroniza estado |
| `PATCH .../{id}/arquivar` | Corpo `{}` | Arquiva somente Tarefa concluída |
| `PATCH .../{id}/desarquivar` | Corpo `{}` | Desarquiva e volta para coluna de concluídas |
| `DELETE /api/v1/implantacao/tarefas/{id}` | Sem corpo | Exclui, audita e recalcula jornada |
| `POST .../{id}/comentarios` | `ComentarioCriarRequest` | Acrescenta comentário |
| `GET /api/v1/implantacao/tarefas/chamados/busca` | `buscar?`, `take?` (1..200), `clienteId?`, `incluirFechados?` (default `false`) | Consulta `tbchamado` (restrita ao cliente quando informado). `buscar` numérico = **prefixo do `Id`, com uma faixa por comprimento** (`591`; `5910-5919`; `59100-59199`; … até 9 dígitos), combinadas com `OR` e com o LIKE do título como `OU`; o `Id` **exato vem primeiro** (`CASE WHEN Id = @termo THEN 0 ELSE 1`) e os demais em ordem **crescente**; sem busca numérica, `Id DESC`. **Chamados com `STATUS "F"` ficam fora por padrão** (comparação ignora caixa e espaços; `STATUS` nulo continua visível) |
| `POST .../{id}/chamados` | `TarefaChamadoRequest` | Vincula chamado legado |
| `DELETE .../{id}/chamados/{chamadoId}` | Sem corpo | Desvincula e limpa referência legada quando aplicável |
| `POST .../{id}/apontamentos` | `ApontamentoCriarRequest` | Cria apontamento e recalcula horas |
| `PUT .../apontamentos/{id}` | `ApontamentoAtualizarRequest` | JWT; o controller calcula admin por `IsInRole("Administrador")`, claim `eh_admin` ou claim `perfil_id = "A"` — o token emitido satisfaz os três, então o admin edita apontamento de outro operador; nos demais, só o próprio apontamento |
| `DELETE .../apontamentos/{id}` | Sem corpo | JWT; a mesma detecção de admin do `PUT` — o admin exclui apontamento de outro operador; nos demais, só o próprio apontamento |
| `GET /api/v1/implantacao/tarefas/{id}/historico` | Sem corpo | Histórico derivado de `tbauditoriaimplantacao` |
| `GET /api/v1/implantacao/dashboard` | `equipe?`, `projetoId?`, `funcaoId?` | Agregados do dashboard **filtrados pelo setor**; `PodeVerFuncao` → `403` para setor não permitido ou operador sem setor (fora do Master) |

Os DTOs estão em [`TarefaDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Implantacao/TarefaDtos.cs). Para apontamentos, o controller calcula `ehAdmin` com `User.IsInRole("Administrador")`, claim `eh_admin = "true"` ou claim `perfil_id = "A"`. O token emitido pelo `AuthService` carrega **os três** (role `Administrador`, `eh_admin` e `perfil_id` **normalizado**), então o administrador é reconhecido: a regra efetiva deixou de ser só "próprio apontamento".

A regra de etapa do projeto é aplicada no serviço, não no controller. `TarefaService.ValidarEtapaFixaAsync` exige `ProjetoEtapaId` quando a tarefa tem projeto (`Etapa do projeto é obrigatória para tarefas de projeto`) e confirma que a etapa pertence a esse projeto (`Etapa do projeto inválida para esta tarefa`). Sem projeto, a validação é ignorada. Na criação, `CriarAsync` rejeita etapa sem projeto (`Não é possível informar uma etapa do projeto sem informar o projeto.`); na atualização, `AtualizarAsync` valida a etapa contra o projeto já persistido, mantém `t.ProjetoId` inalterado — `TarefaAtualizarRequest` não transporta projeto — e grava `t.ProjetoEtapaId = null` quando a tarefa é sem projeto.

**Data de conclusão real: preenchida pelo servidor, nunca pelo cliente.** `TarefaCriarRequest` e `TarefaAtualizarRequest` **não têm** `DataConclusao`; a data é derivada do status/coluna dentro de `TarefaService`. `CriarAsync` nasce a tarefa com a data nula e a registra se ela já nascer `Concluida`. `AtualizarAsync` **preserva** a data na edição comum (inclusive quando só o título muda) e a **limpa** quando a tarefa sai de `Concluida` — pela derivação do status/coluna e pela invariante final do método. `MudarColunaAsync` segue a mesma regra ao mover o card. O campo continua nas respostas (`TarefaDetalhe`, `TarefaResumo`, `CardKanban`), mas é **somente leitura**. Regra coberta por [`TarefaDataConclusaoTests.cs`](../../../../backend/Central_BackEnd.Tests/TarefaDataConclusaoTests.cs).

