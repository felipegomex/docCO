---
id: mod.dashboard
titulo: "Dashboard Integrado"
tipo: modulo
area: dashboard
sistema: central
tags: [dashboard, comercial, leads, propostas, administrativo, financeiro, kpi, contrato, custo]
dono: administracao
status: ativo
revisado_em: "2026-10-29"
fonte: [backend/Central_BackEnd/Controllers/Dashboard, backend/Central_BackEnd/Controllers/ComercialController.cs, backend/Central_BackEnd/Services/Dashboard, backend/Central_BackEnd/Models/Dashboard, frontend/src/app/features/dashboard]
testes: [backend/Central_BackEnd.Tests/DashboardServicosTests.cs]
tokens_estimados: 1700
---

# Dashboard Integrado

## 1. Quando usar este card

KPIs administrativos, financeiros e comerciais, funil de leads, propostas,
custos, orçamentos, contratos e vínculo de lead com cliente.

## 2. Propósito e escopo

Reúne **Comercial**, **Administrativo** e **Financeiro** sobre o legado e as
tabelas novas do funil. Não cobre o dashboard operacional por função
(`/implantacao`, `/suporte`) nem o Kanban de tarefas.

## 3. Rotas do frontend

| URL | Componente | Fonte |
|---|---|---|
| `/dashboard` | `DashboardHomeComponent` | `frontend/src/app/features/dashboard/dashboard.routes.ts` |
| `/dashboard/comercial` | `ComercialComponent` / `LeadsKanbanComponent` | idem + `comercial/comercial.routes.ts` |
| `/dashboard/comercial/propostas` | `PropostasComponent` | `comercial/comercial.routes.ts` |
| `/dashboard/administrativo` | `AdministrativoComponent` | `dashboard.routes.ts` |
| `/dashboard/financeiro` | `FinanceiroComponent` | idem |

`/dashboard` exige `aba.adm.visao-geral`; `/dashboard/financeiro` exige também
`aba.adm.financeiro` (`frontend/src/app/features.routes.ts`).

## 4. Endpoints do backend

| Rotas | Autorização | Fonte |
|---|---|---|
| `GET /dashboard/administrativo/clientes`, `/grupos`, `/contratos`, `/resumo` | `perm:aba.adm.administrativo:Ler` | `backend/Central_BackEnd/Controllers/Dashboard/AdministrativoController.cs` |
| `GET /dashboard/financeiro/kpis`, `/atrasos`; CRUD `/custos`, `/orcamentos` | `perm:aba.adm.financeiro:Ler` | `backend/Central_BackEnd/Controllers/Dashboard/FinanceiroController.cs` |
| CRUD `/dashboard/comercial/leads`, `/oportunidades`; `GET /produtos`, `/kpi` | `perm:aba.adm.comercial:Ler` | `backend/Central_BackEnd/Controllers/ComercialController.cs` |

## 5. Tabelas e entidades

| Classe → tabela | Papel | Fonte |
|---|---|---|
| `TbLead` → `tblead` | Lead do funil | `backend/Central_BackEnd/Models/Dashboard/TbLead.cs` |
| `TbLeadContato` → `tbleadcontato` | Histórico de contato | `backend/Central_BackEnd/Models/Dashboard/TbLeadContato.cs` |
| `TbOportunidade` → `tboportunidade` | Oportunidade | `backend/Central_BackEnd/Models/Dashboard/TbOportunidade.cs` |
| `TbOrcamento` → `tborcamento` | Orçamento | `backend/Central_BackEnd/Models/Dashboard/TbOrcamento.cs` |
| `TbCusto` → `tbcusto`; `TbContratoLegado` → `tbcontrato` | Custo e contrato | `backend/Central_BackEnd/Models/Dashboard/TbCusto.cs` |
| — → `tbcliente`, `tbcliente_grupo`, `tbproduto` | Leitura do legado | `backend/Central_BackEnd/Services/Dashboard/AdministrativoService.cs` |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `AdministrativoService` | Clientes, grupos, contratos e KPI | `backend/Central_BackEnd/Services/Dashboard/AdministrativoService.cs` |
| backend | `FinanceiroService` | KPIs, atrasos, custos e orçamentos | `backend/Central_BackEnd/Services/Dashboard/FinanceiroService.cs` |
| backend | `ComercialService` | Leads, etapas, oportunidades e KPI | `backend/Central_BackEnd/Services/Dashboard/ComercialService.cs` |
| backend | `FlagsDashboard` | Interruptor de faturamento | `backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs` |
| frontend | `AdministrativoService`, `FinanceiroService`, `ComercialService` | HTTP das três abas | `frontend/src/app/features/dashboard/administrativo/administrativo.service.ts` |
| frontend | `LeadsKanbanComponent` | Kanban de leads | `frontend/src/app/features/dashboard/comercial/leads-kanban.component.ts` |

## 7. Regras de negócio

- Etapas fixas: `Prospecção`, `Apresentação`, `Proposta`, `Negócio Fechado`
  (`EtapasComercial.Validas`, `ConstantesDashboard`).
- Lead novo entra em `Prospecção`; etapa inválida devolve `ArgumentException`.
- Oportunidade só é vinculada quando o lead está em `Proposta`
  (`ComercialService`).
- Custo aceita `Salário`, `Imposto`, `Estrutura`, `Outro`; orçamento segue
  `Rascunho → Enviado → Aprovado → Faturado`.
- **Faturamento desligado de propósito**: `FlagsDashboard.FaturamentoLiberado =
  false`; a API devolve zero em vez de projetar receita de `tbcontrato`.
- Origem do lead e tipo de contato têm listas fixas (`OrigensLead.Validas`,
  `TiposContatoLead.Validos`).

## 8. Autorização

- Policies por área: `perm:aba.adm.administrativo:Ler`,
  `perm:aba.adm.financeiro:Ler` e `perm:aba.adm.comercial:Ler`.
- O frontend exige `aba.adm.visao-geral` no pai e `aba.adm.financeiro` na aba
  financeira; o backend revalida.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “adicionar etapa do funil” | `backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs#EtapasComercial` | `frontend/src/app/features/dashboard/models/dashboard.model.ts#ETAPAS_COMERCIAIS` | — (sem teste dedicado) |
| “mover lead de etapa” | `backend/Central_BackEnd/Services/Dashboard/ComercialService.cs#MoverLeadEtapaAsync` | `frontend/src/app/features/dashboard/comercial/leads-kanban.component.ts#LeadsKanbanComponent` | — (sem teste dedicado) |
| “custo” | `backend/Central_BackEnd/Services/Dashboard/FinanceiroService.cs#CriarCustoAsync` | `frontend/src/app/features/dashboard/financeiro/financeiro.component.ts#FinanceiroComponent` | — (sem teste dedicado) |
| “orçamento” | `backend/Central_BackEnd/Services/Dashboard/FinanceiroService.cs#CriarOrcamentoAsync` | `frontend/src/app/features/dashboard/financeiro/financeiro.component.ts#FinanceiroComponent` | — (sem teste dedicado) |
| “KPI administrativo” | `backend/Central_BackEnd/Services/Dashboard/AdministrativoService.cs#ObterKpiAsync` | `frontend/src/app/features/dashboard/administrativo/administrativo.component.ts#AdministrativoComponent` | — (sem teste dedicado) |
| “ligar faturamento” | `backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs#FlagsDashboard` | `frontend/src/app/features/dashboard/models/dashboard.model.ts` | — (decisão registrada) |
| “oportunidade/proposta” | `backend/Central_BackEnd/Services/Dashboard/ComercialService.cs#ListarOportunidadesAsync` | `frontend/src/app/features/dashboard/comercial/propostas.component.ts#PropostasComponent` | — (sem teste dedicado) |

## 10. Contratos que não podem quebrar

- `FaturamentoLiberado = false` é decisão de negócio; não devolver projeção.
- Etapa/status/tipo são strings persistidas; renomear quebra dados.
- Custo e orçamento gravam o operador autor.
- Leitura do legado (`tbcliente`, `tbcliente_grupo`, `tbcontrato`, `tbproduto`) é
  somente leitura, exceto o vínculo de lead a cliente.

## 11. Armadilhas conhecidas

- Compatibilidade 100: nomes de coluna e `CLIENTE_ID` não são renomeáveis.
- KPI comercial conta só leads não arquivados, salvo filtro explícito.
- Vínculo de cliente novo só é aceito em `Negócio Fechado`.

## 12. Testes

- Backend: `DashboardServicosTests` (etapas, custo, orçamento e flag).
- Frontend: sem spec das três abas; cenários em
  `docs/referencia/qa/13-8-dashboard.md`.

## 13. Orçamento de leitura

- Abra primeiro esta ficha; depois o arquivo da seção 9.
- Não abra `docs/referencia/backend/05-4-servicos-principais.md` inteiro.
- Limite: 1 ficha, 4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- Testes de serviço em `DashboardServicosTests`; o faturamento segue bloqueado
  até o Financeiro confirmar a fonte (`FlagsDashboard.FaturamentoLiberado`).
