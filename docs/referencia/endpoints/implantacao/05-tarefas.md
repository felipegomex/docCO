---
id: implantacao.05
titulo: "Tarefas"
tipo: referencia
area: api
sistema: central
tags: [api, implantacao, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/Implantacao
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 1822
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Tarefas

Fonte: [TarefasController.cs](../../../../backend/Central_BackEnd/Controllers/Implantacao/TarefasController.cs). Todas as operações abaixo usam `[Authorize]` e `validacao` no nível de classe.

| Método | Rota | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/api/v1/implantacao/tarefas` | `validacao` | Query equivalente a `TarefaFiltro`: `projetoId`, `equipe`, `responsavelId`, `status`, `prioridade`, `tipo`, `buscar`, `apenasAtrasadas`, `apenasEmAndamento`, `apenasConcluidas`, `apenasVenceHoje`, `incluirArquivadas`, `funcaoId`, `funcaoClassificacao`, `perfilId`, `perfilModo`, `responsavelFuncaoId`, `apenasBloqueadas`, `limite` | `200 List<TarefaResumo>` com `funcaoId`/`funcaoNome`/`funcaoClassificacao`/`etapaTemplateId`/`etapaTemplateNome` e com **`atrasado`** (decisão do backend pela regra única de prazo: `dataEntrega ?? dataPrevisao` vencido, exceto concluída dentro do prazo; cancelada nunca conta); `bloqueada`/`motivoBloqueio` vêm de `tbbloqueio`, não de coluna. **`403` quando `funcaoId` é setor proibido para o operador** (a UI traduz como estado explícito de sem acesso). `limite` é aceito de 1 a 2000, com padrão **500** quando ausente ou fora da faixa; o serviço marca `UltimaListagemTruncada` quando corta. |
| `GET` | `/api/v1/implantacao/tarefas/{id}` | `validacao` | Path `id`; sem body | `200 TarefaDetalhe` ou `404`. |
| `POST` | `/api/v1/implantacao/tarefas` | `validacao` | Body `TarefaCriarRequest` | `201 TarefaDetalhe`; tarefa com projeto exige `ProjetoEtapaId` do mesmo projeto. `400 "Coluna inválida para a função da tarefa."` — a função da tarefa vem do projeto ou, sem projeto, do **criador**, e a coluna precisa estar **visível no quadro** desse setor. O contrato **não tem `DataConclusao`**: a data de conclusão real é preenchida pelo servidor. |
| `PUT` | `/api/v1/implantacao/tarefas/{id}` | `validacao` | Body `TarefaAtualizarRequest` | `200 TarefaDetalhe` ou `404`; mesma validação de coluna do `POST` (`400 "Coluna inválida para a função da tarefa."`). O contrato **não tem `DataConclusao`**: a data de conclusão real é preservada na edição comum e limpa quando a tarefa sai de `Concluida`. |
| `PATCH` | `/api/v1/implantacao/tarefas/{id}/coluna` | `validacao` | Body `TarefaMudarColunaRequest` | `200 TarefaDetalhe`; aplica WIP e sincronizações internas; `400 "Coluna inválida para a função da tarefa."` quando a coluna não está visível no quadro do setor da tarefa. |
| `PATCH` | `/api/v1/implantacao/tarefas/{id}/arquivar` | `validacao` | Sem body | `200 TarefaDetalhe`; somente concluídas. |
| `PATCH` | `/api/v1/implantacao/tarefas/{id}/desarquivar` | `validacao` | Sem body | `200 TarefaDetalhe`; retorna para concluída. |
| `DELETE` | `/api/v1/implantacao/tarefas/{id}` | `validacao` | Path `id`; sem body | `204` ou `404`. |
| `POST` | `/api/v1/implantacao/tarefas/{id}/comentarios` | `validacao` | Body `ComentarioCriarRequest` | `200 ComentarioTarefaResumo`. |
| `GET` | `/api/v1/implantacao/tarefas/chamados/busca` | `validacao` | Query `buscar?`, `take=20`, `clienteId?`, `incluirFechados=false` | `200 List<ChamadoLegadoResumo>`. Com `clienteId`, restringe aos chamados daquele cliente (usado pelo form para casar com o `CLIENTE_ID` do projeto). `buscar` numérico casa por **prefixo do `Id`**, com **uma faixa por comprimento** (`591`; `5910-5919`; `59100-59199`; `591000-591999`; … até 9 dígitos) combinadas com `OR`, e o **LIKE do título continua como `OU`**; não numérico, só o LIKE. **Ordenação:** com busca numérica, o `Id` **exato vem primeiro** (`CASE WHEN Id = @termo THEN 0 ELSE 1`) e os demais em ordem **crescente** de `Id`; sem busca (ou só por título), `Id DESC` (mais recentes primeiro). **Chamados fechados (`STATUS "F"`) ficam fora por padrão** (`incluirFechados=false`): a comparação ignora caixa e espaços e o `STATUS` nulo continua visível; `incluirFechados=true` devolve também os fechados. `take` é normalizado para **1..200** (valor ≤ 0 vira 20). |
| `POST` | `/api/v1/implantacao/tarefas/{id}/chamados` | `validacao` | Body `TarefaChamadoRequest` | `200 TarefaDetalhe` ou `404`. |
| `DELETE` | `/api/v1/implantacao/tarefas/{id}/chamados/{chamadoId}` | `validacao` | Path `id`, `chamadoId` | `200 TarefaDetalhe`; desvincula o chamado. |
| `POST` | `/api/v1/implantacao/tarefas/{id}/apontamentos` | `validacao` | Body `ApontamentoCriarRequest` | `200 ApontamentoResumo`. |
| `PUT` | `/api/v1/implantacao/tarefas/apontamentos/{apontamentoId}` | `validacao` | Body `ApontamentoAtualizarRequest` | `200 ApontamentoResumo` ou `404`; `403` para apontamento de outro operador quando o controller não identifica admin (ele identifica pelo role `Administrador`, pela claim `eh_admin` ou pela claim `perfil_id = "A"` — o token emitido carrega as três). |
| `DELETE` | `/api/v1/implantacao/tarefas/apontamentos/{apontamentoId}` | `validacao` | Path `apontamentoId`; sem body | `204` ou `404`; `403` para apontamento de outro operador nas mesmas condições. |
| `GET` | `/api/v1/implantacao/tarefas/{id}/historico` | `validacao` | Path `id`; sem body | `200 List<HistoricoMovimentacao>` derivado da auditoria. |

⚠️ **Dois eixos de função, lado a lado.** Antes da conversão, `funcaoId` e `setorId` eram nomes diferentes para o mesmo eixo (o do card), e não havia parâmetro para a função do *responsável*:

| Parâmetro | Eixo | Quem usa |
|---|---|---|
| `funcaoId` | **função dona do card** (`TRF_FuncaoId`) — é o recorte do quadro | a tela de Kanban (`carregarBoardPorPerfil` e o filtro por pill) |
| `responsavelFuncaoId` | função de quem **responde** pelo card (via `tboperador.FUNCAO_ID`) | consulta por pessoa; o frontend não envia |

`funcaoClassificacao` continua sendo **aceito e não aplicado** — não é filtro, é atributo da função usado pelo cliente para colorir o badge. A colisão foi resolvida em [`TarefasController.cs`](../../../../backend/Central_BackEnd/Controllers/Implantacao/TarefasController.cs) e documentada no docblock de [`TarefaDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Implantacao/TarefaDtos.cs).

**Autorização de responsáveis:** criação e edição verificam o responsável principal; os itens de `ResponsavelIds` não seguem uma regra uniforme entre os fluxos. Na criação, cada item é apenas consultado quanto à existência e os inexistentes são ignorados; na atualização, quando enviada, cada responsável adicional passa pela autorização de atribuição. O parâmetro não deve ser tratado como uma regra uniforme de propriedade.

**Data de conclusão real — só leitura, preenchida pelo servidor.** `TarefaCriarRequest` e `TarefaAtualizarRequest` **não têm** `DataConclusao`: a data é derivada do status/coluna, no `TarefaService`. A tarefa **nasce** concluída com a data registrada; a **edição comum preserva** a data já registrada; e a data **some** quando a tarefa sai de `Concluida` (a data não "força" mais a conclusão). A resposta continua entregando o campo, apenas para leitura (`TarefaDetalhe`, `TarefaResumo` e `CardKanban`).

