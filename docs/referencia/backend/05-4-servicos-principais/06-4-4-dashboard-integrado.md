---
id: 05-4-servicos-principais.06
titulo: "4.4 Dashboard Integrado"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 808
ficha: docs/modulos/dashboard.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Dashboard Integrado](../../../modulos/dashboard.md).

### 4.4 Dashboard Integrado

| Serviço | Responsabilidade | Dependências | Validações confirmadas | Efeitos colaterais confirmados |
|---|---|---|---|---|
| `ComercialService` (`IComercialService`) | CRUD de leads e oportunidades, arquivamento, catálogo de produtos, KPIs do funil, histórico de contatos e **flag "cliente viu o contato"** | `AppDbContext` | Etapas e respostas validadas contra `ConstantesDashboard`; lead/oportunidade devem existir; tipos de contato fixos em `TiposContatoLead`; `nomeContato` obrigatório; cliente e operador precisam existir; oportunidade só em lead `Proposta` não arquivado; produto precisa existir em `tbproduto` | Mover lead fecha contato; excluir lead apaga oportunidades; `ArquivarLeadAsync`/`DesarquivarLeadAsync` gravam `DataArquivamento`/`MotivoArquivamento`; `ListarLeadsAsync` e `ObterKpiAsync` excluem arquivados por padrão; `CriarOportunidadeAsync` calcula `ValorEstimado = implementação + porLicença × qtde`; enviar proposta muda a etapa para `Proposta` e `NÃO` grava `MotivoPerda`; criar contato atualiza `LEAD_UltimoContato`; excluir contato recompõe `UltimoContato`; `DefinirContatoVisualizadoAsync` grava `UsuarioAlteracao`/`DataAlteracao` sem revalidar origem/cliente; `CriarClienteAsync` faz `INSERT` bruto em `tbcliente` com `MAX(CLIENTE_ID)+1` dentro da transação do fechamento |
| `AdministrativoService` (`IAdministrativoService`) | Base de clientes filtrada, grupos econômicos, contratos e resumo | `AppDbContext` | Somente leitura; filtros e ordenação inválidos são ignorados (sem `400`) | Nenhuma escrita; `ListarClientesAsync` faz **uma** consulta com LEFT JOIN em `tbcontrato` e `tbcliente_grupo`; `ListarGruposAsync` faz **duas** consultas e une os totais em memória. **`Gate de faturamento`**: com `FlagsDashboard.FaturamentoLiberado = false`, `ObterKpiAsync` devolve `FaturamentoMensalContratos = 0m` e `ListarContratosAsync` devolve `ContratoResumo.ValorTotal = 0m`; os demais campos seguem normais. `diasParaReajuste` é `int?` e vale **`null`** quando o contrato não tem `DataProximoReajuste` (nunca `0`) |
| `FinanceiroService` (`IFinanceiroService`) | KPIs, atrasos, custos e orçamentos | `AppDbContext` | Tipo de custo, competência `AAAA-MM` e status de orçamento validados; contrato só conta se `TbContratoLegado.EhAtivo`; vencido só se `DiaFaturamento` ∈ 1..31 e `< hoje.Day` | Grava `DataAprovacao` ao aprovar, limpa ao sair de `Aprovado` e grava `DateTime.Now` ao faturar. **`Gate de faturamento`**: com `FlagsDashboard.FaturamentoLiberado = false`, `ObterKpiAsync` devolve `FaturamentoPrevistoMes`, `FaturamentoAtrasado`, `ResultadoMes` e `ContratosAtrasados` em `0` e **pula o cálculo de `vencidos`** (`new List<TbContratoLegado>()`); `ListarAtrasosAsync` faz **retorno antecipado com lista vazia**. `CustoMes`, `OrcamentosAprovadosPendentes`, `ReajustesProximos90Dias` (data, não dinheiro) e `EvolucaoCustos` **não** são afetados |

