---
id: sec.07-6-componentes-standalone
titulo: "6. Componentes standalone"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 3464
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 6. Componentes standalone

Os componentes Angular localizados declaram `standalone: true`; não há componente de aplicação com `standalone: false`. Não existe um `NgModule` que declare componentes da aplicação. `NgbModule` é importado como coleção de providers, não como declarations.

### 6.1 Componentes roteados

Os componentes roteados e suas URLs estão consolidados nas seções 4.2 a 4.6. `LoginComponent`, `MainLayoutComponent` e `AdminLayoutComponent` são os três componentes diretamente registrados nas rotas raiz.

### 6.2 Componentes internos, modais e dinâmicos

| Componente | Como entra em execução |
|---|---|
| `HeaderComponent`, `SidebarComponent`, `FooterComponent`, `BreadcrumbComponent` | Imports do `MainLayoutComponent` |
| `Jota3dComponent` (`app-jota-3d`) | Compartilhado em [`shared/components/jota-3d/`](../../../frontend/src/app/shared/components/jota-3d/), **sem consumidor na versão atual** (`/chat` virou iframe no `cb36ad3`; o `JotaWidgetComponent` foi removido). Standalone, `OnPush`, inputs signal `estado` (`JotaEstado`), `tamanho` (`sm\|md\|lg`), `tamanhoPx` (`number \| null` → CSS var `--jota-size`), `decorativo`, `modeloUrl` (default `assets/models/jota/jota.glb`) e `posterUrl`; output `falha`; enquanto o gate `JOTA_3D_HABILITADO` for `false` só o poster 2D é exibido e `three.js` nunca é importado. Plano: [`planos/jota-3d/00-PLANO-E-ESFORCO.md`](../../planos/jota-3d/00-PLANO-E-ESFORCO.md); o `.glb` e a ativação do gate seguem pendentes |
| `GlobalSearchComponent` | Import de `HeaderComponent` e `HomeComponent` |
| `PageHeaderComponent` | Import em várias páginas e no shell do Database |
| `UsuarioDropdownComponent` | Import de `KanbanComponent` e `TarefaFormComponent` |
| `ChamadoDropdownComponent` | Import de `TarefaFormComponent`; busca **server-side** no backend com **debounce de 300 ms** — **não há refiltro local por substring** (o backend é a única fonte do filtro e a tela renderiza direto o que veio da API). Recebe `[clienteId]` (restringe a busca ao `CLIENTE_ID` do projeto selecionado; na troca, **limpa a busca** e recarrega), `[rotulos]` (mapa id → título dos chamados já vinculados, para o chip mostrar o título de um chamado que a busca esconde, ex.: `STATUS "F"`) e `[selecionados]`/`(selecaoChange)`. **Guarda de corrida** (uma resposta antiga não sobrescreve a lista de uma busca mais nova) e **recarga ao abrir** o painel (a lista exibida era a da última busca) |
| `ProjetoCardComponent` | Import de `ProjetosComponent`; incorpora `ProjetoEtapaCardComponent` |
| `ProjetoEtapaCardComponent` | Componente interno de cards do Projeto |
| `ProjetoEtapaModalComponent` | Import e template de `ProjetoDetalheComponent` |
| `ProjetoRetornoDialogComponent` | Import e template de `ProjetoDetalheComponent` |
| `OnboardingSectionComponent` | Import de `OnboardingCapituloComponent` |
| `DbGlobalSearchComponent` | Import de `DatabaseShellComponent` |
| `DbQueryBuilderComponent` | Import de `DbConsultasComponent` |
| `DbSincronizacaoComponent` | Import de `DbDiferencasComponent` |
| `DbScriptsCorrecaoComponent` | Import de `DbSincronizacaoComponent` (aba de comparação) |
| `DbProceduresResultadoComponent` | Import de `DbSincronizacaoComponent` (modo procedures) |
| `AgendaEventoModalComponent` | `import()` dinâmico e `NgbModal.open` |
| `DbProcedureModalComponent` | `import()` dinâmico e `NgbModal.open` |
| `DbTriggerModalComponent` | `import()` dinâmico e `NgbModal.open` |
| `LeadModalComponent` | `NgbModal.open` a partir de `LeadsKanbanComponent`; usa `NgbActiveModal` e `styleUrls` do partial `dash-kpis.scss`. Também hospeda o checkbox "cliente visualizou o contato" e o bloco de criação de oportunidade |
| `DashboardToastComponent` | Montado por cada página do módulo e alimentado por `DashboardToastService` |
| `ContatoLeadModalComponent` | `NgbModal.open` a partir de `LeadsKanbanComponent` e `LeadModalComponent`; usa `NgbActiveModal` |
| `FerramentaEmbedComponent` | Painel burro de iframe (seletor `app-ferramenta-embed`) hospedado pelo `FerramentaWorkspaceComponent`; recebe `ferramenta/slot/podeComparar/emTelaCheia` e emite `comparar/fechar/trocar/alternarTelaCheia`; monta o iframe só em browser com `bypassSecurityTrustResourceUrl` (`loading=lazy`, `referrerpolicy`, `allow=fullscreen`, sem `sandbox` amplo); toolbar com recarregar, comparar, trocar ferramenta, tela cheia interna, abrir em nova aba e fechar; banner heurístico (timeout 10 s sem `load`) quando o site pode bloquear incorporação |

### 6.3 Telas do Dashboard Integrado (Home, Comercial, Administrativo e Financeiro)

Tabela de inventário e comportamento confirmado no código:

| Componente | Responsabilidade confirmada | Observações de implementação |
|---|---|---|
| `DashboardHomeComponent` | Visão geral: KPIs **comercial** (leads no funil, negócios fechados, taxa de conversão, pipeline aberto) e **administrativo** (clientes ativos, contratos ativos) + atalhos para os três módulos | **Sem indicadores financeiros.** O bloco "Financeiro" (previsto / em atraso / custos / resultado), o card "Faturamento mensal" e o card "Reajustes em 90 dias" foram **removidos** porque todo valor de faturamento é uma projeção de `tbcontrato` ainda não liberada (`FlagsDashboard.FaturamentoLiberado = false`). O `forkJoin` do `ngOnInit` chama **apenas** `ComercialService.obterKpi()` e `AdministrativoService.obterResumo()` — não há `FinanceiroService`, `kpiFinanceiro` nem import de `FinanceiroKpi`. Descrição do page-header: “Visão integrada do funil comercial e da operação administrativa.” O atalho Financeiro é o único condicional (`*ngIf="ehAdministrador()"`) e tem a descrição “Custos mensais, orçamentos e evolução.” |
| `LeadsKanbanComponent` | Kanban de leads por etapa (CDK Drag & Drop), busca, filtro de arquivados, KPIs do funil e modal de lead/contato | `alternarContatoVisualizado(lead, event)` faz **update otimista com rollback**, espelhando `moverLeadEtapa`: aplica o novo valor no signal, chama `PATCH /leads/{id}/contato-visualizado` e, no erro, reverte e mostra toast. O botão tem `stopPropagation` para não abrir o modal do card. Estilo `.dash-kanban__visto` (verde "Viu" / cinza "Não viu") em `leads-kanban.component.scss`. O rodapé do card soma `valorOportunidades` (total de `tboportunidade` do lead). O toggle `Ver arquivados / Ver ativos` (`verArquivados`, recarrega com `somenteArquivados`) separa a visão principal do histórico; o card tem botão arquivar (pede motivo via prompt → `PATCH /leads/{id}/arquivar`) e, na visão de arquivados, desarquivar. |
| `LeadModalComponent` | Cadastro/edição do lead, histórico de contatos, **checkbox "cliente visualizou o contato"** e **oferta de produtos** | O checkbox usa a classe `.dash-vinculo-check` do partial `dash-kpis.scss` e é enviado como `contatoVisualizado` no `PUT /leads/{id}`. O bloco "Oportunidades / Produtos" só aparece em `@if (lead)` (edição) **e somente quando o lead está na etapa `Proposta`** (nas demais, aviso orienta mover o lead). O checklist lista o catálogo `tbproduto` (`listarProdutos`), com estado vazio ("cadastre no legado") e erro visível se o GET falhar (nunca silêncio); o botão Ofertar desabilita sem produtos; cada produto marcado gera **1 `POST /oportunidades`** com valor implementação + valor por PA/licença + estimativa PA + observação (`ValorEstimado` calculado no backend). A lista exibe produto, composição de valores, observação e motivo de perda, com exclusão por linha. A timeline de contatos exibe `nomeContato · data/hora`. |
| `PropostasComponent` | Ciclo da proposta: enviar, marcar visualizada e registrar/limpar resposta (SIM/NÃO/AGUARDANDO) | Lista via `GET /oportunidades` com filtros locais. O empty state aponta para o Kanban (`routerLink="/dashboard/comercial"`), porque a criação de oportunidade passou a ocorrer no modal do lead. |
| `AdministrativoComponent` | **Somente a base de clientes**: 3 KPIs, barra de filtros e tabela única | Reescrito com signals. Restam os KPIs **Clientes ativos**, **Contratos ativos** e **Licenças em contratos ativos**; foram removidos o card "Faturamento mensal", o card "Reajustes em 90 dias" e o alerta "N reajuste(s) em até 30 dias". Por isso o componente **não consome mais `listarContratos()`**: saíram o signal `contratos`, o computed `reajustesCriticos`, o método `classeReajuste` e os imports `ContratoResumo`, `formatarData` e `formatarMoeda`; a tela chama apenas `/resumo`, `/clientes` e `/grupos`. O estilo `.dash-adm__alerta` em `administrativo.component.scss` ficou **órfão** e só volta a ter uso se o alerta for reintroduzido. Filtros em `campoFiltro`, `busca`, `situacao`, `grupoId`, `ordenacao`, com constantes `CAMPOS_FILTRO_CLIENTE`, `SITUACOES_FILTRO_CLIENTE` e `ORDENACOES_CLIENTE` exportadas pelo próprio componente. `placeholderBusca()` muda conforme o campo escolhido; `totalLicencas()` soma as licenças da lista exibida; `temFiltroAtivo()` controla o botão Limpar. Tabela: `# \| Cliente \| CNPJ \| Situação \| Licenças \| Grupo`. Estilos em uso: `.dash-adm__filtros`, `.dash-adm__filtro`, `.dash-adm__filtro-campo`, `.dash-adm__filtro-busca`, `.dash-adm__limpar`, `.dash-adm__contador`, `.dash-adm__licencas`, `.dash-adm__num`, `.dash-adm__bloco`, `.dash-adm__sub`. |
| `FinanceiroComponent` | `adminGuard`: 2 KPIs, gráfico de evolução e **CRUD de custos e orçamentos** | Restam os KPIs **Orçamentos aprovados pendentes** e **Reajustes em 90 dias**; foram removidos "Faturamento previsto (mês)", "Faturamento atrasado", "Custo do mês", "Resultado do mês" e o card combinado "Contratos atrasados · reajustes 90d", que virou só "Reajustes em 90 dias". A tabela "Faturamentos em atraso" também saiu, junto com o signal `atrasos`, a chamada `listarAtrasos()` e o import `AtrasoDto`. **Mantidos de propósito:** gráfico de evolução, tabela de Custos e tabela de Orçamentos, porque vêm de `tbcusto`/`tborcamento`, tabelas que o sistema gerencia e escreve (ambas vazias hoje). A coluna "Faturamento" dos Orçamentos também permanece: é `tborcamento.dataFaturamento`, **dado nosso**, e não a projeção do legado — não confundir com o `diaFaturamento` do contrato. Descrição do page-header e dica em `header-nav.config.ts`: “Custos mensais, orçamentos e evolução.” |

Os tipos espelham os DTOs do backend em [`dashboard.model.ts`](../../../frontend/src/app/features/dashboard/models/dashboard.model.ts): `LeadResumo.contatoVisualizado`, `LeadResumo.arquivado/dataArquivamento/motivoArquivamento`, `ContatoVisualizadoRequest`, `ProdutoResumo`, `OportunidadeResumo.produtoId/produtoDescricao/valorImplementacao/valorPorLicenca/qtdeLicencas/motivoPerda`, `ContatoLeadResumo.nomeContato`, `ClienteResumo.licencas/grupoId/grupoNome`, `ClienteGrupo`, `CampoFiltroCliente`, `SituacaoFiltroCliente`, `OrdenacaoCliente`, `ContratoResumo.diaFaturamento`, `AtrasoDto.diaFaturamento` e `ContratoResumo.diasParaReajuste: number | null` (anulável de propósito — contrato sem data de reajuste não é “0 dias”).

> Distinção importante: `diaFaturamento` (dia do mês do contrato legado) **não** é o mesmo campo que `dataFaturamento` de `OrcamentoResumo` (data real, em `tborcamento`, tabela gerida pelo JCA).
>
> Com `FlagsDashboard.FaturamentoLiberado = false`, `AdministrativoKpi.faturamentoMensalContratos`, `FinanceiroKpi.faturamentoPrevistoMes/faturamentoAtrasado/resultadoMes/contratosAtrasados` e `ContratoResumo.valorTotal` chegam **zerados** ao frontend; nenhum componente os exibe hoje. A separação por tela é o que impede os números de aparecerem — os campos continuam no tipo. Detalhes em [`03-REGRAS-NEGOCIO.md`](../regras-negocio/INDEX.md).

### 6.4 Componente removido e item que permanece órfão

O `AdminDashboardComponent` do frontend e o `AdminDashboardService` que ele consumia foram **removidos em 2026-10-20** (Fase 0): estavam em `features/admin/pages/dashboard/` e `features/admin/services/`, sem rota e sem import ativo.

O que **permanece órfão** é o **backend**: `AdminDashboardController` (`GET /api/v1/admin/dashboard`) e `AdminDashboardService` continuam existindo e registrados no DI, mas nenhuma rota nem item de menu do frontend os consome. O endpoint segue documentado em [`07-SERVICES-BACKEND.md`](../backend.md) e em [`05-ENDPOINTS.md`](../endpoints.md).

`JotaWidgetComponent` foi **removido no `cb36ad3`** (2026-11-03): o widget global não existe mais no layout; a página `/chat` é iframe do AnythingLLM (`ChatIframeComponent`).

