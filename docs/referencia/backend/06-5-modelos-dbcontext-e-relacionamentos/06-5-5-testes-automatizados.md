---
id: 06-5-modelos-dbcontext-e-relacionamentos.06
titulo: "5.5 Testes automatizados"
tipo: referencia
area: backend
sistema: central
tags: [backend, modelos, dbcontext, efcore]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2734
ficha: docs/modulos/database-explorer.md
---
> Parte de [5. Modelos, DbContext e relacionamentos](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 5.5 Testes automatizados

`backend/Central_BackEnd.Tests` é o **único projeto de teste do repositório** (xunit 2.9.2 + FluentAssertions 6.12.1, `net8.0`, com `Microsoft.EntityFrameworkCore.InMemory` disponível).

| Arquivo | Casos | Cobre |
|---|---:|---|
| [`ValidadoresAcessoTests.cs`](../../../../backend/Central_BackEnd.Tests/ValidadoresAcessoTests.cs) | 28 | CNPJ (normalizar, vazio→`null`, 14 dígitos com/sem máscara, tamanho errado, formatar), IPv4 (9 casos) e porta (6 casos) |
| [`CifradorTests.cs`](../../../../backend/Central_BackEnd.Tests/CifradorTests.cs) | 14 | Roundtrip AES-256-GCM, envelope com 4 partes e prefixo `v1`, **IV aleatório** (mesma senha → ciphertext diferente), retrocompatibilidade com texto puro, vazio, e as 3 rejeições de chave (ausente, não-base64, tamanho errado); **rejeição dos 4 marcadores de placeholder** (`Theory`, verificado mesmo quando o valor parece base64) e **`GerarChaveBase64()`** devolvendo 32 bytes válidos e aleatórios, usáveis de imediato; `CifradorEmBranco` preserva texto e reporta `EhCifradoDeVerdade = false` |
| [`TemSenhaTests.cs`](../../../../backend/Central_BackEnd.Tests/TemSenhaTests.cs) | 5 | **Regressão do sombreamento**: `TemSenha` espelha a `SenhaCifrada` da própria classe, e por reflexão confirma que `AcessoBase` e `AcessoActyonWeb` **não** declaram `SenhaCifrada` |
| [`KanbanFuncaoTests.cs`](../../../../backend/Central_BackEnd.Tests/KanbanFuncaoTests.cs) | 33 | **Matriz de permissão por função** (admin, suporte, perfil `F`, operador comum, sem função, origem sem função, operador desconhecido), **bloqueio** (motivo obrigatório, reabrir atualiza em vez de duplicar, fechar registra quem e quando, gargalos ordenados e filtrados por função, função da origem de tarefa/projeto/etapa) e **quadro** (`403` ao mover card de outra função, coluna de outra função, coluna inativa, movimento de projeto, delegação ao `TarefaService`, `truncado`, união tarefa+projeto, filtro por tipo e card bloqueado com motivo/idade) |
| [`TicketAcessoTests.cs`](../../../../backend/Central_BackEnd.Tests/TicketAcessoTests.cs) | 23 | Gate de senha: emissão/validação de ticket `VISUALIZAR`/`EDITAR`, escopo por operador/empresa, uso único, expiração e motivos de recusa |
| [`AcessosUnidadeTests.cs`](../../../../backend/Central_BackEnd.Tests/AcessosUnidadeTests.cs) | 9 | **Contrato de identidade única**: criar sem `clienteId` = erro (sem cadastro manual), empresa inexistente/inativa = erro, duplicado por `ClienteId` devolve a existente, identidade sempre do ERP (mudou na `tbcliente` → a tela muda), lista/busca por identidade do ERP, cadeia de fallback `Fantasia → RazaoSocial → "Cliente {id}"`, edição só situação/observação e recusa sem ticket |
| [`TbDocSchemaTests.cs`](../../../../backend/Central_BackEnd.Tests/TbDocSchemaTests.cs) | 3 | Gravação/leitura por `Caminho`, unicidade lógica (UPSERT substitui, não duplica) e modelo com índice único em `Caminho`/tamanhos (`300`/`8`) — docs-api fase 1 |
| [`DocumentosServiceTests.cs`](../../../../backend/Central_BackEnd.Tests/DocumentosServiceTests.cs) | 5 | `ListarIndiceAsync` sem corpo, `ObterPorCaminhoAsync` com/sem prefixo `docs/`, `null` (→ `404`) para vazio e travessia `..`, e hash divergente marca regravação — docs-api fase 2 |
| [`RegrasPrazoTests.cs`](../../../../backend/Central_BackEnd.Tests/RegrasPrazoTests.cs) | 27 | **Contrato da regra única de prazo**: matriz de tarefa (13 casos: prazo vencido, concluída dentro/depois do prazo, concluída sem data, cancelada, prazo hoje/futuro, sem prazo e `DataEntrega` mandando sobre `DataPrevisao`), "vence hoje" de tarefa (5), matriz de projeto (7), atraso de etapa (referência = `DataFimReal` ou hoje; etapa sem prazo = 0) e o caso que **amarra a expressão do EF** (`OndeAtrasadas`) à regra do serviço, linha a linha |
| [`PerfilOperadorTests.cs`](../../../../backend/Central_BackEnd.Tests/PerfilOperadorTests.cs) | 28 | **PERFIL_ID lido em um lugar só**: normalização (trim + maiúsculas, vazio → `null`), rótulo a partir da letra normalizada, `EhAdmin` com `SE_ADMIN` e com a letra, e as integrações — operador com `"a"`/ `" A "` **é admin no Kanban** e o token carrega `perfil_id` normalizado com o rótulo canônico |
| [`TarefaChamadoBuscaTests.cs`](../../../../backend/Central_BackEnd.Tests/TarefaChamadoBuscaTests.cs) | 13 | **Busca de chamados do dropdown "atrelar chamado"**: prefixo do `Id` com o **exato primeiro** e os demais em ordem crescente, exclusão do que não começa com o prefixo, `STATUS "F"` **oculto por padrão** e presente com `incluirFechados=true`, variação de caixa/espaço no status, `STATUS` nulo visível, busca por título, recorte por `clienteId`, limite de `take` e ordem `Id DESC` sem termo — mais um caso que **trava a tradução para SQL Server** (`ToQueryString()`: o InMemory não prova tradução), exigindo o `LIKE` de título, o `CASE` do `Id` exato e as 7 faixas do prefixo |
| [`TarefaDataConclusaoTests.cs`](../../../../backend/Central_BackEnd.Tests/TarefaDataConclusaoTests.cs) | 6 | **Data de conclusão real é do servidor**: o contrato de criação/edição **não expõe** `DataConclusao`, a tarefa nascida em coluna concluída registra a data, a nascida em "A FAZER" não tem data, a edição comum **preserva** a data, sair da coluna concluída a **limpa** e concluir pelo Kanban a registra |
| [`PermissoesCargoHierarquiaTests.cs`](../../../../backend/Central_BackEnd.Tests/PermissoesCargoHierarquiaTests.cs) | 8 | **Hierarquia da matriz**: Master gerencia cargos comuns e não o próprio Master; Tech Lead gerencia o cargo do próprio setor abaixo do nível e não outro setor, outro Tech Lead nem Master; comum não gerencia; as duas listas de cargos gerenciáveis; e salvar a matriz **preserva** a linha por função (`PEC_FuncaoId = 5`) |
| [`IdeiasCiaaLeitorTests.cs`](../../../../backend/Central_BackEnd.Tests/IdeiasCiaaLeitorTests.cs) | 12 | **Leitor do CIAA**: vê a solicitação de colega do setor em modo leitura (sem comentar/editar/transicionar/aprovar), não vê outro setor e outro setor não vê a do CIAA; criador continua criador; a listagem inclui o setor CIAA, respeita `SomenteMinhas`, filtra por prioridade e não vaza outro setor; o detalhe abre para o leitor e devolve `null` para outro setor; o nome do criador é exposto ao leitor |
| **Total** | **214** | 28 + 14 + 5 + 33 + 23 + 9 + 3 + 5 + 27 + 28 + 13 + 6 + 12 + 8. Contagem por caso executado: cada `[InlineData]` de `[Theory]` é um caso. **Tabela parcial**: o total cobre apenas os arquivos listados acima; as demais suítes do módulo de Ideias, as duas de docs-api e as demais não entram nele |

A **Fase 07** acrescentou **15 casos** em [`WhatsAppKanbanServiceDeskTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppKanbanServiceDeskTests.cs): filtro de empresa, etiqueta, prioridade, investigação e as duas dimensões do SLA (cada um conferindo o `TotalCount`), combinação de filtros sem duplicar/perder card, total e paginação do recorte, situação de SLA inválida (`ArgumentException`), card completo/pendente, etiquetas de vários cards em consulta agrupada e três provas de **tradução para SQL Server** (`ToQueryString` sem `OPENJSON`). A **Fase 08** acrescentou **16 casos** em [`WhatsAppAdminContatoTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppAdminContatoTests.cs): filtros de busca/identificação/empresa/telefone (isolados e combinados), paginação, agregação sem N+1, contato sem atendimento, `@lid`, histórico por contato, `403` sem perfil administrativo e tradução da consulta para SQL Server. A **Fase 09** acrescentou **13 casos** em [`WhatsAppRealtimeTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppRealtimeTests.cs): rota e corpo do `typing` no gateway, payload sem conteúdo sensível, falha de publicação que não desfaz a operação, publicação **pós-commit**, autorização de `join` (operador ativo e atendimento existente), `nota` que **não** chama o gateway e debounce de 3 s por atendimento+operador. A **Fase 4 (métricas por etiqueta)** acrescentou **2 casos** em [`WhatsAppMetricasEtiquetaTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppMetricasEtiquetaTests.cs) (volume/tempo médio/taxa de reabertura por etiqueta, incluindo inativa; e lista vazia sem vínculos), e a **Fase 6** completou a cobertura do plano com os outros 10 casos (§4.5.1.3).

As correções de 2026-10-26 acrescentaram **4 casos** (tradução de etiquetas com `IN (...)` sem `OPENJSON`, `ORDER BY`/`OFFSET`/`FETCH` e segunda página em admin/contatos, e substituição do conjunto de etiquetas) e as features **F4/F5/F6** acrescentaram **12 casos** (baseline 855 → **871** na suíte inteira; filtro `FullyQualifiedName~WhatsApp` 362 → **378**): [`WhatsAppFinalizacaoTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppFinalizacaoTests.cs) (**2** — eventos `finalizacao` e `arquivamento`, incluindo o job), [`WhatsAppVistoTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppVistoTests.cs) (**5** — transição para Visto, limpeza de responsável/não lidas, recusa em estado terminal, reabertura pela mensagem e `ReabrirAsync` manual, e o recorte de métricas) e [`WhatsAppGrupoAtendimentoTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppGrupoAtendimentoTests.cs) (**5** — grupo/solicitante no primeiro ciclo, um atendimento ativo por grupo, novo ciclo com novo solicitante, fallback de `SolicitanteId` e `solicitar-privado`). Medição local de 2026-10-26: **871 no projeto** e **378 no filtro WhatsApp**, **0 falhas**; `npx ng build` concluído com sucesso. O `scripts/validate.ps1` deste commit **passou verde** (segredos + build + testes + ng build).

A **Feature 2** (nome do grupo no card) acrescentou [`WhatsAppGrupoSyncTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppGrupoSyncTests.cs) e um caso de lista em `WhatsAppGrupoAtendimentoTests`; medição local de 2026-10-27: **876 testes no projeto**, **0 falhas**, `validate.ps1` verde. Sem deploy nesta entrega.

Em **2026-11-03**, `dotnet test --filter "FullyQualifiedName~Ideias"` fechou em **200 aprovados**. Entraram 5 casos de atribuição/validação de responsável na aprovação (`IdeiasKanbanTests`), 2 de filtro de prioridade (`IdeiasCiaaLeitorTests`, agora 12) e 1 de auditoria `NEGADO` (`IdeiasControllerNegacaoTests`).

```powershell
Set-Location .\backend\Central_BackEnd.Tests
dotnet test
Set-Location ..\..
```

⚠️ **`scripts/validate.ps1` executa `dotnet test` no passo 5/7** (a suíte de `backend/Central_BackEnd.Tests`) e `ng test` no passo 6/7, além de `dotnet build` (Debug), `ng build` e o gate de docs (7/7). Se o passo 5 ou 6 falhar, o gate fica vermelho e o commit/deploy não deve seguir. Do lado do frontend, há arquivos `*.spec.ts` (inventário de **177 casos** — medição de 2026-10-26/27, não reexecutada nesta revisão) e `frontend/karma.conf.js` existe; o pipeline executa `ng test` em modo headless (`ChromeHeadlessNoSandbox`).

