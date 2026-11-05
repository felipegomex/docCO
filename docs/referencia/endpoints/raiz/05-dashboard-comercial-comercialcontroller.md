---
id: raiz.05
titulo: "Dashboard Comercial (`ComercialController`)"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 1397
ficha: docs/modulos/dashboard.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

## Dashboard Comercial (`ComercialController`)

Fonte: [ComercialController.cs](../../../../backend/Central_BackEnd/Controllers/ComercialController.cs), na raiz da pasta `Controllers/` com rota base `/api/v1/dashboard/comercial`. Cada método declara a própria política de limite: `EnableRateLimiting("leitura")` nos `GET` e `EnableRateLimiting("validacao")` nas escritas.

> **Controller canônico do comercial:** `Controllers/ComercialController.cs`. Havia um segundo controller com a **mesma** rota em `Controllers/Dashboard/ComercialController.cs`, o que produzia `AmbiguousMatchException` (HTTP 500) em todas as rotas declaradas nos dois. O duplicado foi apagado; qualquer arquivo novo sob `Controllers/Dashboard` precisa ser conferido contra `Controllers/` para não repetir a colisão (os dashboards administrativo e financeiro, hoje em `Controllers/Dashboard/`, estão em [`dashboard.md`](../dashboard.md)).

### Leads (`/leads`)

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/leads` | Query `statusEtapa?`, `busca?`, `incluirArquivados?`, `somenteArquivados?` | `200 List<LeadResumo>` (inclui `oportunidades`, `valorOportunidades`, `ultimoTipoContato`, `contatoVisualizado`, `arquivado`, `dataArquivamento` e `motivoArquivamento`). Arquivados são **excluídos por padrão**. |
| `GET` | `/leads/{id}` | Path `id` | `200 LeadResumo` ou `404`. |
| `POST` | `/leads` | Body `LeadCriarRequest`: `titulo?`, `clienteId?`, `operadorId?`, `statusEtapa?`, `anotacoes?` | `201 LeadResumo`; etapa inválida → `400 { mensagem }`. |
| `PUT` | `/leads/{id}` | Body `LeadAtualizarRequest` | `200 LeadResumo` ou `404`. O campo `contatoVisualizado` é **opcional** e só é aplicado quando informado. |
| `PATCH` | `/leads/{id}/etapa` | Body `LeadMoverEtapaRequest`: `statusEtapa` | `200 LeadResumo`; mover para `Negócio Fechado` apenas grava `UltimoContato` (não cria contrato). |
| `PATCH` | `/leads/{id}/contato-visualizado` | Body `ContatoVisualizadoRequest`: `visualizado` (bool) | `200 LeadResumo` ou `404`. Marca/desmarca manualmente o flag "cliente viu o contato", grava auditoria e **não revalida** origem nem cliente. |
| `PATCH` | `/leads/{id}/arquivar` | Body `LeadArquivarRequest`: `motivo?` (até 500) | `200 LeadResumo` ou `404`. Arquiva negócio fechado ou proposta não convertida (sai da visão principal, mantém histórico). |
| `PATCH` | `/leads/{id}/desarquivar` | Sem body | `200 LeadResumo` ou `404`. |
| `DELETE` | `/leads/{id}` | Path `id` | `204` ou `404`; oportunidades do lead são excluídas em cascata. |
| `GET` | `/leads/{id}/contatos` | Path `id` | `200 List<ContatoLeadResumo>` ordenados por `DataContato` decrescente (inclui `nomeContato`). |
| `POST` | `/leads/{id}/contatos` | Body `ContatoLeadCriarRequest`: `nomeContato` (obrigatório), `tipo`, `dataContato?`, `observacao?` | `201 ContatoLeadResumo`; nome vazio ou tipo inválido → `400`; lead inexistente → `404`. Atualiza `LEAD_UltimoContato`. |

### Produtos, contatos e oportunidades

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/produtos` | Query `busca?` | `200 List<ProdutoResumo>` (`id`, `descricao`, `obs`) do catálogo legado `tbproduto` (somente leitura). |
| `DELETE` | `/contatos/{id}` | Path `id` | `204` ou `404`; recompõe `LEAD_UltimoContato` com o maior `DataContato` restante. |
| `GET` | `/oportunidades` | Query `etapa?`, `busca?`, `apenasPropostas?` | `200 List<OportunidadeResumo>` (inclui `produtoId`, `produtoDescricao`, `valorImplementacao`, `valorPorLicenca`, `qtdeLicencas`, `motivoPerda`); `apenasPropostas=true` filtra `dataPropostaEnviada != null`. É a fonte de dados da aba Propostas. |
| `GET` | `/oportunidades/{id}` | Path `id` | `200 OportunidadeResumo` ou `404`. |
| `POST` | `/oportunidades` | Body `OportunidadeCriarRequest`: `leadId`, `produtoId?`, `valorImplementacao?`, `valorPorLicenca?` (PA = licença), `qtdeLicencas?`, `valorEstimado?` (fallback), `etapa?`, `observacao?` | `201 OportunidadeResumo`; lead inexistente, lead arquivado ou lead fora da etapa `Proposta` → `400`; produto inexistente → `400`. `ValorEstimado = implementação + porLicença × qtde`. Chamado pelo checklist do modal do lead (1 linha por produto). |
| `PUT` | `/oportunidades/{id}` | Body `OportunidadeAtualizarRequest` (+ `motivoPerda?`) | `200 OportunidadeResumo` ou `404`; informar valores recalcula `ValorEstimado`. |
| `PATCH` | `/oportunidades/{id}/proposta` | Body `PropostaAtualizarRequest`: `dataPropostaEnviada?`, `propostaVisualizada?`, `respostaCliente?`, `dataResposta?`, `motivoPerda?` | `200 OportunidadeResumo`; enviar proposta move a oportunidade para a etapa `Proposta`; `respostaCliente` vazio limpa resposta e motivo; `NÃO` + `motivoPerda` = proposta não convertida. |
| `DELETE` | `/oportunidades/{id}` | Path `id` | `204` ou `404`. |

### KPI comercial

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/kpi` | Sem body | `200 ComercialKpiDto` (leads ativos — arquivados excluídos —, funil por etapa, propostas, pipeline, ticket médio, conversão). `ValorPipeline` e `ValorFechado` somam `tboportunidade`. |

As rotas `/oportunidades*` foram **mantidas** mesmo com a tela removida: a aba Propostas, o KPI comercial e o `valorOportunidades` do card do Kanban dependem delas.

