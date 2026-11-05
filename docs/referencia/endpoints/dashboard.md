---
title: "Catálogo de endpoints — pasta Dashboard"
description: "Controllers de backend/Central_BackEnd/Controllers/Dashboard: dashboards integrados administrativo e financeiro sob /api/v1/dashboard."
date: "2026-10-18"
language: "pt-BR"
status: "canônico"
---

# Endpoints — Controllers/Dashboard

Pasta `backend/Central_BackEnd/Controllers/Dashboard`: **2 classes** e **14 operações** `[Http*]`, conferidas em 2026-10-18 pela contagem recursiva dos atributos. O comercial do Dashboard Integrado (`ComercialController`, 20 operações) fica na raiz da pasta `Controllers` e está em [raiz.md](raiz/INDEX.md); a colisão de rota que motivou a remoção do duplicado lá está registrada na mesma seção — qualquer arquivo novo sob `Controllers/Dashboard` precisa ser conferido contra `Controllers/` na raiz. Índice do catálogo em [`endpoints.md`](../endpoints.md); seções por pasta: [`raiz.md`](raiz/INDEX.md), [`implantacao.md`](implantacao/INDEX.md) e [`database.md`](database.md).

Suma por controller (operações `[Http*]`):

| Controller | Operações | Rota base |
|---|---:|---|
| `AdministrativoController` / Dashboard Integrado | 4 | `/api/v1/dashboard/administrativo` |
| `FinanceiroController` / Dashboard Integrado | 10 | `/api/v1/dashboard/financeiro` |
| **Total** | **14** | — |

Os dois controllers sob `/api/v1/dashboard` têm `[ApiVersion("1.0")]` e exigem apenas JWT; o financeiro é restrito a `Administrador`. Ambos aplicam `EnableRateLimiting("validacao")` no controller inteiro.

## Administrativo (`/api/v1/dashboard/administrativo`) — JWT

Fonte: [AdministrativoController.cs](../../../backend/Central_BackEnd/Controllers/Dashboard/AdministrativoController.cs).

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/clientes` | Query `busca?`, `campo?` (`todos`\|`cliente`\|`cnpj`\|`grupo`), `situacao?` (`ativo`\|`inativo`), `grupoId?`, `ordenacao?` (`cliente`\|`licencas-desc`\|`licencas-asc`\|`id`) | `200 List<ClienteResumo>` da legada `tbcliente`, em **uma** consulta com LEFT JOIN em `tbcontrato` (soma `QTDE_LICENCA`) e `tbcliente_grupo` (nome do grupo). Cliente sem contrato continua na lista com `0` licença. |
| `GET` | `/grupos` | Sem body | `200 List<ClienteGrupoDto>` (`id`, `nome`, `clientes`, `licencas`) da legada `tbcliente_grupo`. São **duas consultas** (grupos e licenças por `GRUPO_ID`) unidas em memória. |
| `GET` | `/contratos` | Query `busca?` | `200 List<ContratoResumo>` da legada `tbcontrato` (somente leitura, com `diaFaturamento` e `diasParaReajuste` **anulável**). `diasParaReajuste` é `null` quando o contrato não tem `DATA_PROXIMO_REAJUSTE` — nunca `0`. `valorTotal` vem `0` enquanto o interruptor de faturamento estiver desligado. A tela Administrativa não consome este endpoint. |
| `GET` | `/resumo` | Sem body | `200 AdministrativoKpiDto` (clientes/contratos ativos, licenças, reajustes 90d, leads no funil, e `faturamentoMensalContratos` que vem `0` enquanto o interruptor estiver desligado). |

Filtros e ordenação de `/clientes` são validados no serviço; valor fora do conjunto é ignorado (não gera `400`). `campo`, `situacao` e `ordenacao` valem apenas quando enviados explicitamente.

## Financeiro (`/api/v1/dashboard/financeiro`) — JWT + Administrador

Fonte: [FinanceiroController.cs](../../../backend/Central_BackEnd/Controllers/Dashboard/FinanceiroController.cs).

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/kpis` | Sem body | `200 FinanceiroKpiDto` (custo do mês, orçamentos pendentes, reajustes 90d e série `evolucaoCustos`). `faturamentoPrevistoMes`, `faturamentoAtrasado`, `resultadoMes` e `contratosAtrasados` **devolvem `0`** enquanto o interruptor de faturamento estiver desligado. |
| `GET` | `/atrasos` | Sem body | `200 List<AtrasoDto>` de contratos **ativos** com faturamento do mês corrente já vencido. **Com o interruptor de faturamento desligado, a lista volta vazia por retorno antecipado.** O legado não tem data de faturamento: quando liberado, o serviço considera vencido o contrato cujo `diaFaturamento` (1–31) é menor que o dia de hoje, e `diasAtraso = hoje.Day - diaFaturamento`; contrato sem `DiaFaturamento` nunca aparece. |
| `GET` | `/custos` | Query `mesInicio?`, `mesFim?` (formato `AAAA-MM`) | `200 List<CustoResumo>`. |
| `POST` | `/custos` | Body `CustoCriarRequest`: `tipo`, `valor`, `mesReferencia`, `dataPagamento?`, `observacao?` | `201 CustoResumo`; tipo/competência inválidos → `400 { mensagem }`. |
| `PUT` | `/custos/{id}` | Body `CustoAtualizarRequest` | `200 CustoResumo` ou `404`. |
| `DELETE` | `/custos/{id}` | Path `id` | `204` ou `404`. |
| `GET` | `/orcamentos` | Query `status?` | `200 List<OrcamentoResumo>`. |
| `POST` | `/orcamentos` | Body `OrcamentoCriarRequest`: `clienteId?`, `descricao?`, `valor`, `status?` | `201 OrcamentoResumo`; status vazio vira `Rascunho`. |
| `PUT` | `/orcamentos/{id}` | Body `OrcamentoAtualizarRequest`: `clienteId?`, `descricao?`, `valor?`, `status`, `dataFaturamento?` | `200 OrcamentoResumo`; transições regem `DataAprovacao`/`DataFaturamento` (aprovar grava; sair de `Aprovado` limpa; `Faturado` grava agora). |
| `DELETE` | `/orcamentos/{id}` | Path `id` | `204` ou `404`. |

Erros de argumento (`ArgumentException`) retornam `400 { mensagem }` nos dois controllers.

> **Interruptor de faturamento (desligado).** `ConstantesDashboard.FaturamentoLiberado` é `false`: os campos derivados de `tbcontrato` (`faturamentoMensalContratos`, `ContratoResumo.valorTotal`, `faturamentoPrevistoMes`, `faturamentoAtrasado`, `resultadoMes`, `contratosAtrasados` e a lista de `/atrasos`) **continuam no contrato e no JSON**, mas vêm zerados. **Religar é trocar a constante para `true` em [`ConstantesDashboard.cs`](../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs)**: nenhuma rota, migration ou DTO muda. Campos **não** afetados: `custoMes`, `orcamentosAprovadosPendentes`, `reajustesProximos90Dias`, `evolucaoCustos` e todo o CRUD de custos e orçamentos (`tbcusto`/`tborcamento`, tabelas do próprio sistema). Justificativa e números de referência em [`03-REGRAS-NEGOCIO.md`](../regras-negocio/INDEX.md).
