---
id: mod.ideias
titulo: "Ideias, Melhorias e Erros"
tipo: modulo
area: ideias
sistema: central
tags: [ideias, melhorias, erros, sugestao, kanban, aprovacao, ia, especificacao]
dono: produto
status: ativo
revisado_em: "2026-11-03"
fonte: [backend/Central_BackEnd/Controllers/IdeiasController.cs, backend/Central_BackEnd/Services/Ideias, backend/Central_BackEnd/Models/Ideias, backend/Central_BackEnd/Dtos/Ideias, frontend/src/app/features/ideias]
testes: [backend/Central_BackEnd.Tests/IdeiasCicloDeVidaTests.cs, backend/Central_BackEnd.Tests/IdeiasKanbanTests.cs, backend/Central_BackEnd.Tests/IdeiasAutorizacaoTests.cs, backend/Central_BackEnd.Tests/IdeiasControllerNegacaoTests.cs]
tokens_estimados: 1950
---

# Ideias, Melhorias e Erros

## 1. Quando usar este card

Registrar ideia/melhoria/relato de erro, lista e filtros, detalhe e comentários,
transição de status, análise por IA, especificação, aprovação com criação de
tarefa no Kanban, permissão de gestor e leitor Ciaa.

## 2. Propósito e escopo

Recebe sugestões, melhorias e erros, conduz o ciclo até aprovação e, quando
aprovada, cria **uma** tarefa no Kanban existente. Não cobre o Kanban
(`kanban.md`) nem a IA de chat (`chat-conhecimento.md`).

## 3. Rotas do frontend

| URL | Componente | Fonte |
|---|---|---|
| `/ideias` | redirect para `/ideias/lista` | `frontend/src/app/features/ideias/ideias.routes.ts` |
| `/ideias/lista` | `ListaIdeiasComponent` | idem |
| `/ideias/nova` e `/nova/:tipo` | `NovaIdeiaComponent` | idem |
| `/ideias/:id` | `DetalheIdeiaComponent` | idem |

A rota pai exige uma de `aba.sistema.ideias`, `aba.ciaa.ideias` ou
`aba.implantacao.ideias` (`frontend/src/app/features.routes.ts`).

## 4. Endpoints do backend

| Rotas | Autorização | Fonte |
|---|---|---|
| `GET/POST /ideias`, `GET/PATCH /ideias/{id}` | `[Authorize]` + escopo | `backend/Central_BackEnd/Controllers/IdeiasController.cs` |
| `DELETE /ideias/{id}` | ADMIN ou gestor (geral ou da função do criador) | idem — **exclusão lógica**: arquiva (botão "Arquivar"), responde 204/403/404; negação audita `NEGADO` |
| `POST /ideias/{id}/mensagens`, `/transicoes`, `/arquivar`, `/analises`, `/aprovar` | escopo | idem |
| `PATCH /ideias/{id}/prioridade-oficial`, `DELETE /ideias/{id}/vinculo-kanban` | idem | idem |
| `PUT/POST /ideias/{id}/especificacao`, `/aprovar`, `/exige-revisao` | idem | idem |
| `GET /ideias/minhas-permissoes` | `[Authorize]` | idem |
| `POST /ideias/seed` | `[Authorize]` | `backend/Central_BackEnd/Controllers/IdeiasSeedController.cs` |

`GET /ideias` aceita `tipo`, `status`, `de`, `ate`, `busca`, `incluirArquivadas`,
`somenteMinhas` e `prioridade` (`IdeiaFiltro.Prioridade`). A tela de lista expõe
busca, tipo, status, prioridade, período, "Somente minhas" e o checkbox de
arquivadas.

`POST /ideias/{id}/aprovar` recebe `funcaoResponsavelId`, `responsavelId`
(string, opcional) e `justificativa` (`AprovacaoRequest`).

## 5. Tabelas e entidades

| Classe → tabela | Papel | Fonte |
|---|---|---|
| `Ideia` → `tbideia` | Registro e status | `backend/Central_BackEnd/Models/Ideias/Ideia.cs` |
| `MensagemIdeia` → `tbmensagemideia`; `HistoricoStatusIdeia` → `tbhistoricostatusideia` | Comentários e trilha | `backend/Central_BackEnd/Models/Ideias/MensagemIdeia.cs` |
| `AnaliseIdeia` → `tbanaliseideia` | Resultado da IA | `backend/Central_BackEnd/Models/Ideias/AnaliseIdeia.cs` |
| `EspecificacaoIdeia` → `tbespecificacaoideia` | Especificação | `backend/Central_BackEnd/Models/Ideias/EspecificacaoIdeia.cs` |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `IdeiaService` | CRUD, comentário, transição, arquivo | `backend/Central_BackEnd/Services/Ideias/IdeiaService.cs` |
| backend | `TransicoesStatusIdeia` | Matriz de status válidos | `backend/Central_BackEnd/Services/Ideias/TransicoesStatusIdeia.cs` |
| backend | `IdeiaKanbanService` | Aprova e cria uma tarefa | `backend/Central_BackEnd/Services/Ideias/IdeiaKanbanService.cs` |
| backend | `IdeiaAnaliseService` | Análise por IA | `backend/Central_BackEnd/Services/Ideias/IdeiaAnaliseService.cs` |
| backend | `IdeiaEspecificacaoService` | Salvar/aprovar/revisar | `backend/Central_BackEnd/Services/Ideias/IdeiaEspecificacaoService.cs` |
| backend | `IdeiasPermissaoService` | Gestor, leitor Ciaa e escopo | `backend/Central_BackEnd/Services/Ideias/IdeiasPermissaoService.cs` |
| frontend | `IdeiasService`, `ListaIdeiasComponent`, `DetalheIdeiaComponent` | HTTP, lista e ações | `frontend/src/app/features/ideias/ideias.service.ts` |

## 7. Regras de negócio

- Tipos `Ideia`, `Melhoria`, `RelatoErro`; prioridade `Baixa`, `Media`, `Alta`
  (`backend/Central_BackEnd/Models/Ideias/Ideia.cs`).
- Status: `Recebida`, `EmDiscussao`, `AguardandoEsclarecimentos`,
  `EmAnaliseTecnica`, `AprovadaParaDesenvolvimento`, `NaoAprovada`,
  `Implementada` — mesma fonte.
- Transição inválida é recusada por `TransicoesStatusIdeia.PodeIr`.
- Aprovação cria **uma única** tarefa; repetir devolve a já vinculada (índice
  único filtrado) — `IdeiaKanbanService#AprovarAsync`.
- Aprovar aceita `ResponsavelId` opcional: o operador precisa existir e estar
  ativo (`SeAtivo == "S"`). Se `FuncaoResponsavelId` também vier e divergir do
  `FuncaoId` do responsável, devolve `400` ("O responsavel escolhido nao
  pertence a funcao responsavel informada."). Sem função informada, usa a função
  do responsável; o fallback final é `ideia.FuncaoRelacionadaId`.
- Sem escolha de responsável, a tarefa nasce **sem** responsável na coluna da
  função; com escolha, nasce atribuída. A auditoria `APROVACAO` grava
  `ResponsavelId`.
- Tarefa e vínculo no mesmo `SaveChangesAsync`; não fica “aprovada sem tarefa”.
- Aprovar exige coluna compatível no Kanban; sem coluna é erro de negócio.
- Especificação tem estado próprio (`Rascunho`, `Aprovada`, `Obsoleta`).
- **Exclusão é lógica**: `ExcluirAsync` marca `Arquivada`, `DataArquivamento`,
  `MotivoArquivamento = "Exclusao administrativa"` e `UsuarioAlteracao/DataAlteracao`.
  Nenhum registro satélite (mensagens, especificações, análises, histórico) é
  apagado; a tarefa do Kanban também permanece. Audita `ARQUIVAR` e devolve `true`.
- Filtro de prioridade compara `PrioridadeOficial ?? PrioridadeSugerida`, então
  uma prioridade oficial define o resultado mesmo quando difere da sugerida.
- Negação em endpoint de ação é auditada como `NEGADO` (mensagem e código).

## 8. Autorização

- `[Authorize]` em tudo; o **escopo** deriva do papel no backend: gestor de
  função (`tbgestor`) e leitor Ciaa (`IdeiasPermissaoService#ObterAsync`).
- Leitura sem permissão responde **404** (não revela existência); ação sem
  permissão responde **403** e grava `NEGADO` via `NegarAsync` no
  `IAuditoriaImplantacaoService` (`{ Mensagem, Codigo }`).
- `NegarAsync` cobre Atualizar, Comentar, Transicionar, PrioridadeOficial,
  Arquivar, Analisar, Aprovar, DesvincularKanban, as três ações de
  especificação e Excluir.
- Excluir: ADMIN ou gestor geral/da função do criador; `404` quando a
  solicitação não existe.
- `GET /ideias/minhas-permissoes` alimenta a UI.
- A rota `/ideias` usa `permissaoGuard` com as três abas; o backend é a fonte.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “criar/editar ideia” | `backend/Central_BackEnd/Services/Ideias/IdeiaService.cs#CriarAsync` | `frontend/src/app/features/ideias/pages/nova/nova-ideia.component.ts#NovaIdeiaComponent` | `backend/Central_BackEnd.Tests/IdeiasCicloDeVidaTests.cs` |
| “transição de status” | `backend/Central_BackEnd/Services/Ideias/TransicoesStatusIdeia.cs#PodeIr` | `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts#DetalheIdeiaComponent` | `backend/Central_BackEnd.Tests/IdeiasCicloDeVidaTests.cs` |
| “aprovar e criar tarefa” | `backend/Central_BackEnd/Services/Ideias/IdeiaKanbanService.cs#AprovarAsync` | `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts#DetalheIdeiaComponent` | `backend/Central_BackEnd.Tests/IdeiasKanbanTests.cs` |
| “desvincular do Kanban” | `backend/Central_BackEnd/Services/Ideias/IdeiaKanbanService.cs#CancelarVinculoAsync` | `frontend/src/app/features/ideias/ideias.service.ts#IdeiasService` | `backend/Central_BackEnd.Tests/IdeiasKanbanTests.cs` |
| “análise por IA” | `backend/Central_BackEnd/Services/Ideias/IdeiaAnaliseService.cs#AnalisarAsync` | `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts#DetalheIdeiaComponent` | `backend/Central_BackEnd.Tests/IdeiasAnaliseIaTests.cs` |
| “especificação” | `backend/Central_BackEnd/Services/Ideias/IdeiaEspecificacaoService.cs#SalvarAsync` | `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts#DetalheIdeiaComponent` | `backend/Central_BackEnd.Tests/IdeiasEspecificacaoTests.cs` |
| “permissão gestor/Ciaa” | `backend/Central_BackEnd/Services/Ideias/IdeiasPermissaoService.cs#ObterAsync` | `frontend/src/app/features/ideias/pages/lista/lista-ideias.component.ts#ListaIdeiasComponent` | `backend/Central_BackEnd.Tests/IdeiasAutorizacaoTests.cs` |
| “excluir solicitação (arquiva)” | `backend/Central_BackEnd/Services/Ideias/IdeiaService.cs#ExcluirAsync` / `#PodeExcluirAsync` | `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts#excluir` | `backend/Central_BackEnd.Tests/IdeiasCicloDeVidaTests.cs#Excluir_arquiva_a_solicitacao_e_preserva_os_registros_satelites` |
| “conversa/timeline” | `backend/Central_BackEnd/Services/Ideias/IdeiaService.cs#ObterAsync` | `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts#DetalheIdeiaComponent` | `backend/Central_BackEnd.Tests/IdeiasCicloDeVidaTests.cs` |

## 10. Contratos que não podem quebrar

- Enums são `byte` e persistidos como número; reordenar quebra dados.
- “Uma ideia, uma tarefa”: índice único filtrado garante idempotência.
- Não gravar vínculo sem tarefa nem tarefa sem vínculo; aprovação não pode criar
  card em coluna invisível.
- `minhas-permissoes` é o contrato da UI; não replicar a regra no cliente.
- Exclusão **não** apaga registros: `DELETE /ideias/{id}` arquiva. Não
  reintroduzir delete físico nem remoção de satélites.
- Ação negada = 403 auditado; leitura negada = 404.
- Aprovação com responsável só aceita operador **ativo** e da função informada;
  sem escolha, a tarefa nasce sem atribuição.

## 11. Armadilhas conhecidas

- `System.Text.Json` sem conversor escreve enum como inteiro; o contrato JSON
  depende disso (`IdeiasContratoJsonTests`).
- Compatibilidade SQL Server do legado tem teste próprio.
- A coluna do Kanban precisa existir antes de aprovar.

## 12. Testes

- `IdeiasCicloDeVidaTests`, `IdeiasKanbanTests`, `IdeiasAutorizacaoTests`,
  `IdeiasCiaaLeitorTests`, `IdeiasAnaliseIaTests`, `IdeiasEspecificacaoTests` e
  `IdeiasControllerNegacaoTests`.
- `dotnet test --filter "FullyQualifiedName~Ideias"`: **200 aprovados**
  (2026-11-03).
- QA: `docs/referencia/qa/15-10-ideias-melhorias-e-erros.md`.

## 13. Orçamento de leitura

Abra esta ficha; depois só o arquivo da seção 9 e o teste. Limite: 1 ficha,
4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- Seed manual/guardado (`Ideias:Seed:Habilitado` + recusa de produção); listagem
  com spec de UI.
