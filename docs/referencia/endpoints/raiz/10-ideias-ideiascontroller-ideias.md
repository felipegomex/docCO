---
id: raiz.10
titulo: "Ideias (`IdeiasController`, `/ideias`)"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 766
ficha: docs/modulos/ideias.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Ideias, Melhorias e Erros](../../../modulos/ideias.md).

## Ideias (`IdeiasController`, `/ideias`)

Fonte: [IdeiasController.cs](../../../../backend/Central_BackEnd/Controllers/IdeiasController.cs). Base `/api/v1/ideias`; todo acesso exige autenticação; o escopo de leitura é derivado de `tbgestor` no servidor e o corpo da requisição nunca define autor. **16 operações**, batendo com a contagem do código. Leitura sem permissão responde 404; ação sem permissão responde 403 e é auditada como `NEGADO` em todos os endpoints de ação, inclusive `Excluir`.

| Método | Rota | Quem pode | Observação |
|---|---|---|---|
| GET | `/ideias` | criador, gestor da função ou leitor do CIAA | filtros: `tipo`, `status`, `de`, `ate`, `busca`, `incluirArquivadas`, `somenteMinhas`, `prioridade`, `page`, `pageSize`; prioridade compara `PrioridadeOficial ?? PrioridadeSugerida`; o leitor do CIAA vê as solicitações cujo criador é de `FUNCAO_ID = 5` |
| GET | `/ideias/{id}` | criador, gestor da função ou leitor do CIAA | **404** também quando existe e não é seu (não revela existência); o leitor abre em modo somente leitura |
| POST | `/ideias` | autenticado | autor vem do claim; não existe `criadorId` no corpo |
| PATCH | `/ideias/{id}` | criador, nas fases da matriz | título, descrição, função relacionada |
| POST | `/ideias/{id}/mensagens` | criador ou gestor | append-only, sem edição |
| POST | `/ideias/{id}/transicoes` | gestor | corpo traz a **operação**; o destino vem da matriz |
| PATCH | `/ideias/{id}/prioridade-oficial` | gestor | |
| POST | `/ideias/{id}/arquivar` | gestor | arquivamento **lógico**; motivo obrigatório |
| POST | `/ideias/{id}/analises` | gestor | análise por IA, sincrona (até ~60 s) |
| POST | `/ideias/{id}/aprovar` | gestor | corpo `AprovacaoRequest`: `funcaoResponsavelId?`, `responsavelId?` (string) e `justificativa`. Cria uma tarefa no Kanban e grava o vínculo; com `responsavelId`, exige operador ativo (`SE_ATIVO='S'`) e, se a função informada divergir do `FuncaoId` dele, devolve **400** ("O responsavel escolhido nao pertence a funcao responsavel informada."). Sem função, usa a do responsável (fallback `FuncaoRelacionadaId`); sem responsável, a tarefa nasce sem atribuição. Auditoria `APROVACAO` grava `ResponsavelId` |
| DELETE | `/ideias/{id}` | **Administrador ou gestor** (geral ou da função do criador) | **exclusão lógica**: arquiva a solicitação (`Arquivada`, `DataArquivamento`, `MotivoArquivamento = "Exclusao administrativa"`) e audita `ARQUIVAR`; **nao apaga** satélites (`tbmensagemideia`, `tbhistoricostatusideia`, `tbanaliseideia`, `tbespecificacaoideia`). Responde **204** (sucesso), **403** (nem admin, nem gestor da ideia — autor sem gestão não exclui; a negação grava `NEGADO`) e **404** (inexistente). A tarefa do Kanban vinculada **não** é apagada — fica órfã no quadro. Fora da matriz de operações: excluir é manutenção do cadastro, não etapa do fluxo |
| DELETE | `/ideias/{id}/vinculo-kanban` | gestor | desfaz só o vínculo; 404 quando a tarefa está fechada |
| PUT | `/ideias/{id}/especificacao` | gestor | cria/edita versão do documento técnico; nunca sobrescreve versão aprovada |
| POST | `/ideias/{id}/especificacao/aprovar` | gestor | aprova a versão vigente |
| POST | `/ideias/{id}/especificacao/exige-revisao` | gestor | marca a versão vigente para revisão |
| GET | `/ideias/minhas-permissoes` | autenticado | ajuda de UI (`ehGestor`, `ehLeitor`, `podeVerCriador`, `gestores`, `aviso`) — não é concessão. `podeVerCriador` é `ehGestor` **ou** `ehLeitor`; o aviso de "não administra" não aparece para o leitor do CIAA |

