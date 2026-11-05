---
id: sec.13-12-dashboard-integrado-contra-o-banco-legado-compat-100
titulo: "12. Dashboard Integrado contra o banco legado (compat 100)"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 3175
ficha: docs/modulos/dashboard.md
---
> Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 12. Dashboard Integrado contra o banco legado (compat 100)

O banco legado `dbBUSINESS_HML` estÃ¡ em **`compatibility_level 100` (SQL Server 2008)**. As seÃ§Ãµes 12.1 a 12.6 descrevem as classes de falha que aparecem sempre nessa situaÃ§Ã£o e sÃ£o fÃ¡ceis de reconhecer pela mensagem. As seÃ§Ãµes 12.7 e 12.8 sÃ£o falha **lÃ³gica** do mesmo mÃ³dulo, nÃ£o do `compat 100`: estÃ£o aqui porque a origem dos dados Ã© a mesma (`tbcontrato`).

### 12.1 `AmbiguousMatchException` â€” HTTP 500 em todas as rotas do Comercial

**Sintoma:** 500 em `GET /api/v1/dashboard/comercial/leads`, `GET /leads/{id}`, `POST`/`PUT`/`PATCH`/`DELETE /leads`, todas as `/oportunidades` e `GET /kpi`. No log: `AmbiguousMatchException: Multiple actions matched route`.

**Causa:** dois controllers com a **mesma** template de rota `api/v{version:apiVersion}/dashboard/comercial` â€” `Controllers/ComercialController.cs` e `Controllers/Dashboard/ComercialController.cs`. Toda rota declarada nos dois fica ambÃ­gua no `ActionSelector` e o runtime devolve 500 em vez de 404/405.

**Como confirmar:**

```powershell
rg -n 'Route\("api/v\{version:apiVersion\}/dashboard' .\backend\Central_BackEnd\Controllers
```

Cada prefixo de rota deve aparecer em **um** arquivo.

**SoluÃ§Ã£o:** o duplicado foi apagado; o controller canÃ´nico Ã© `Controllers/ComercialController.cs` (17 operaÃ§Ãµes, `IHostEnvironment` para detalhe de erro em Development e `EnableRateLimiting` por endpoint â€” `"leitura"` nos GET, `"validacao"` nas escritas). Antes de criar um controller sob `Controllers/Dashboard`, verifique se a rota jÃ¡ nÃ£o existe em `Controllers/`.

### 12.2 `Invalid column name` â€” todos os endpoints que leem `tbcontrato`

**Sintoma:** 500 com `Invalid column name 'DATA_FATURAMENTO'` (ou erro equivalente) em `GET /administrativo/clientes`, `/administrativo/contratos`, `/administrativo/resumo`, `/financeiro/kpis` e `/financeiro/atrasos`. Uma Ãºnica coluna errada no espelho derruba **todos** os endpoints que dependem de `tbcontrato`.

**Causa:** `TbContratoLegado` mapeava `DATA_FATURAMENTO`, coluna que **nÃ£o existe** em `tbcontrato`. O schema real tem `DIA_FATURAMENTO smallint NULL` â€” o **dia do mÃªs** do faturamento, nÃ£o uma data.

**SoluÃ§Ã£o:** `TbContratoLegado.DiaFaturamento` (`short?`), propagado para `ContratoResumo.DiaFaturamento` e `AtrasoDto.DiaFaturamento` e para `diaFaturamento` no frontend. O atraso Ã© calculado em `FinanceiroService.EhVencido`: o mÃªs corrente estÃ¡ vencido quando `DiaFaturamento` âˆˆ 1..31 e Ã© menor que o dia de hoje, com `DiasAtraso = hoje.Day - DiaFaturamento`.

**PrevenÃ§Ã£o:** ao mapear uma tabela legada, valide cada coluna contra o dicionÃ¡rio real do banco antes de confiar no nome "Ã³bvio". Um espelho errado derruba o mÃ³dulo inteiro, nÃ£o sÃ³ um endpoint.

### 12.3 `InvalidCastException: Int16 â†’ Decimal`

**Sintoma:** 500 com `InvalidCastException: Unable to cast object of type 'System.Int16' to type 'System.Decimal'` ao ler `tbcontrato`.

**Causa:** `QTDE_LICENCA` Ã© **`smallint`** no banco e estava mapeada como `decimal?` (com `HasPrecision(10,2)` no `AppDbContext`).

**SoluÃ§Ã£o:** `TbContratoLegado.QtdeLicenca` Ã© `short?` e a entity **nÃ£o** aplica `HasPrecision` nessa coluna. A soma de licenÃ§as do Administrativo usa `Sum(t => (int?)(t.QtdeLicenca ?? 0))` justamente para absorver o `smallint` sem CAST implÃ­cito quebrando.

### 12.4 `Sintaxe incorreta prÃ³xima a '$'` â€” `OPENJSON` em compat 100

**Sintoma:** 500 com mensagem de sintaxe perto de `$` (o `OPENJSON(@p)` gerado pelo EF) em queries que filtram uma lista de ids.

**Causa:** `compatibility_level 100` **nÃ£o tem `OPENJSON`**. O EF Core 8 traduz `lista.Contains(x.Id)` para `OPENJSON(@p)`. O padrÃ£o atingido antes foi `ComercialService.ListarLeadsAsync` (agregava oportunidades e contatos filtrando por `leadIds.Contains(...)`) e `TarefaService` (2 pontos ao validar chamados legados).

**Solução (regra permanente do projeto):** agregar ou carregar a tabela pequena **inteira** e filtrar em memÃ³ria. Alternativa para chave Ãºnica: subquery correlacionada (`EXISTS`) ou `IN` com subquery, como em `ProjetoService`. ReferÃªncias do padrÃ£o: comentÃ¡rios "Compatibilidade SQL 2008 (compat 100)" em `DashboardService`, `TarefaService`, `ColunaKanbanService` e `AgendaService`.

**Nível declarado no provider (correção do SLA do WhatsApp).** O `Program.cs` registra o SQL Server com `sql => sql.UseCompatibilityLevel(NivelCompatibilidadeSqlServer)` nos **dois** ramos — Development com `Database:UseSqlServer = true` e ambiente publicado —, com `NivelCompatibilidadeSqlServer = 100`. Com o nível informado, o EF Core 8 volta a traduzir `Contains` de lista como `IN (...)` e deixa de emitir `OPENJSON(... '$')`. O caso que expôs a falta foi o SLA: `CalcularSlaAsync` engolia a `SqlException` no `try/catch` e o SLA simplesmente **não aparecia** — o sintoma era dado ausente na tela, não `500`. Ao investigar SLA que não aparece, procure `SqlException`/`OPENJSON` no log do backend e confirme a declaração do provider antes de suspeitar do cálculo. A correção é só de provider: não há migration nem alteração de schema.

**Armadilha:** o provider InMemory **nÃ£o reproduz** esse erro. Se a tela funcionou em memÃ³ria, a query ainda pode estourar no SQL Server real. Sempre valide contra SQL Server.

### 12.5 Soma dentro de soma â€” erro 130

**Sintoma:** 500 com erro **130** (*"Incorrect number of rows in the subquery"* / subquery retorna nÃºmero incorreto de linhas) ao carregar os grupos econÃ´micos.

**Causa:** somar `QTDE_LICENCA` direto na projeÃ§Ã£o de `tbcliente_grupo` geraria `SUM` aninhado dentro de outro `SUM`, que o SQL Server recusa.

**SoluÃ§Ã£o:** `AdministrativoService.ListarGruposAsync` faz **duas consultas** (grupos com a contagem de clientes, e licenÃ§as agrupadas por `GRUPO_ID` via `tbcontrato`â†’`tbcliente`) e une os totais em memÃ³ria por dicionÃ¡rio.

### 12.6 Cliente sem contrato sumindo da lista

**Sintoma:** a tabela de clientes da tela Administrativa nÃ£o mostra todos os clientes de `tbcliente`.

**Causa:** uso de JOIN interno em `tbcontrato` para somar licenÃ§as.

**SoluÃ§Ã£o:** `ListarClientesAsync` usa **LEFT JOIN** em `tbcontrato` e em `tbcliente_grupo`. O LEFT JOIN Ã© obrigatÃ³rio: cliente sem contrato precisa continuar na lista exibindo `0` licenÃ§a. No homolog havia 6 clientes nessa situaÃ§Ã£o.

### 12.7 Reajuste sem data aparece como "0 dias"

**Sintoma:** o alerta de reajustes lista uma enxurrada de contratos "0 dias" que **nÃ£o tÃªm data de reajuste nenhuma**, e quase nenhum dos itens Ã© real. No homolog, o alerta mostrava **102 itens**, dos quais **101 eram falsos** (contratos sem `DATA_PROXIMO_REAJUSTE`) e apenas **1 real** (contrato 163 / GESTART, 23 dias).

**Causa:** `AdministrativoService.ListarContratosAsync` fazia `dias ?? 0`. `dias` era anulÃ¡vel (o contrato pode nÃ£o ter data), e o `?? 0` colapsava "sem data" em "vencendo hoje". Como o filtro da tela testava `dias >= 0 && dias <= 30`, **todo** contrato sem data entrava como se fosse um reajuste imediato.

**SoluÃ§Ã£o:** o DTO passou a devolver **`null`**, nÃ£o `0`:
- `ContratoResumo.DiasParaReajuste` Ã© `int?` ([`DashboardDtos.cs`](../../../backend/Central_BackEnd/Dtos/Dashboard/DashboardDtos.cs));
- `ContratoResumo.diasParaReajuste` Ã© `number | null` em [`dashboard.model.ts`](../../../frontend/src/app/features/dashboard/models/dashboard.model.ts);
- o serviÃ§o sÃ³ calcula quando `DataProximoReajuste.HasValue`; `Kpi.ReajustesProximos90Dias` jÃ¡ exigia `HasValue` e nÃ£o era afetado.

**Como confirmar o dado bruto:**

```sql
SELECT
  SUM(CASE WHEN DATA_PROXIMO_REAJUSTE IS NULL THEN 1 ELSE 0 END) AS sem_data,
  SUM(CASE WHEN DATA_PROXIMO_REAJUSTE IS NOT NULL THEN 1 ELSE 0 END) AS com_data
FROM tbcontrato;
```

ReferÃªncia de homolog: 185 contratos, 101 com `diasParaReajuste` `null` e 1 dentro da faixa 0â€“30 dias.

**Regra ao reintroduzir qualquer filtro de reajuste (frontend ou service):** testar `diasParaReajuste !== null` / `dias.HasValue` **antes** de comparar com faixa numÃ©rica. Reverter para `?? 0` reintroduz os 101 falsos positivos. O alerta em si foi removido da tela Administrativa junto com os cards de faturamento; o endpoint `GET /administrativo/contratos` continua existindo.

**Armadilha relacionada:** com o interruptor de faturamento desligado (`FlagsDashboard.FaturamentoLiberado = false`), zero no KPI **nÃ£o** significa carteira vazia â€” significa "informaÃ§Ã£o nÃ£o liberada". NÃ£o investigue como se fosse falha de cÃ¡lculo; veja a seÃ§Ã£o 12.8.

### 12.8 Faturamento zerado "do nada"

**Sintoma:** `GET /dashboard/financeiro/kpis` e `GET /dashboard/administrativo/resumo` respondem `200`, mas com `faturamentoPrevistoMes`, `faturamentoAtrasado`, `resultadoMes`, `contratosAtrasados` e `faturamentoMensalContratos` **todos em `0`**, e `GET /dashboard/financeiro/atrasos` volta com lista vazia â€” mesmo com 125 contratos ativos na base.

**Causa:** **nÃ£o Ã© bug.** `ConstantesDashboard.FaturamentoLiberado` estÃ¡ `false` por decisÃ£o do usuÃ¡rio: a informaÃ§Ã£o de faturamento ainda nÃ£o foi liberada. Os serviÃ§os devolvem `0`/lista vazia em vez de publicar a projeÃ§Ã£o `QTDE_LICENCA Ã— VALOR_UNITARIO` derivada de `tbcontrato`, que o legado nÃ£o confirma (nÃ£o hÃ¡ coluna de valor total nem de data de faturamento, e `EhAtivo` inclui `SITUACAO = 'Entregue'`, status de entrega e nÃ£o de cobranÃ§a).

**Como confirmar:**

```powershell
rg -n 'FaturamentoLiberado' .\backend\Central_BackEnd
```

O mesmo flag aparece em `AdministrativoService.ObterKpiAsync`, `AdministrativoService.ListarContratosAsync` (`ValorTotal`), `FinanceiroService.ObterKpiAsync` e `FinanceiroService.ListarAtrasosAsync`.

**O que NÃƒO Ã© afetado:** `reajustesProximos90Dias` (Ã© **data**, nÃ£o dinheiro), `custoMes`, `orcamentosAprovadosPendentes`, `evolucaoCustos` e todo o CRUD de `tbcusto`/`tborcamento` (tabelas que o sistema gerencia e escreve). Se esses tambÃ©m vierem zerados **e** as tabelas estiverem vazias, isso Ã© falta de lanÃ§amento, nÃ£o o interruptor.

**Como religar:** trocar `false` â†’ `true` em [`ConstantesDashboard.cs`](../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs) e republicar. **Do lado da API nenhuma outra mudanÃ§a Ã© necessÃ¡ria** (sem migration, sem ajuste de DTO, sem mudanÃ§a de rota). AtenÃ§Ã£o: os componentes de tela tambÃ©m foram enxugados e **nÃ£o exibem** esses campos hoje; restaurar os cards/tabelas faz parte do religamento (a home estÃ¡ em [`06-COMPONENTES-FRONTEND.md`](../../referencia/frontend.md) e a razÃ£o em [`03-REGRAS-NEGOCIO.md`](../../referencia/regras-negocio/INDEX.md)). Valor de referÃªncia medido com o interruptor ligado, para reconhecer o nÃºmero quando voltar: R$ 259.454,08 de faturamento mensal e R$ 163.970,03 atrasado em 68 contratos.

### 12.9 Checklist rÃ¡pido do mÃ³dulo

| VerificaÃ§Ã£o | Como |
|---|---|
| Nenhuma rota duplicada | `rg -n 'Route\("api/v\{version:apiVersion\}/dashboard' .\backend\Central_BackEnd\Controllers` â€” um arquivo por prefixo |
| Nenhum `Contains` de coleÃ§Ã£o local em query | `rg -n '\.Contains\(' .\backend\Central_BackEnd\Services` e confirme que cada ocorrÃªncia estÃ¡ em memÃ³ria, nÃ£o em IQueryable |
| Colunas do espelho existem | consultar `INFORMATION_SCHEMA.COLUMNS` de `tbcontrato`, `tbcliente`, `tbcliente_grupo` |
| Migration do flag aplicada | `SELECT * FROM __EFMigrationsHistory WHERE MigrationId = '20260928182047_LeadContatoVisualizado'` e `COL_LENGTH('tblead','LEAD_ContatoVisualizado')` |
| Colunas do funil com produtos aplicadas | `COL_LENGTH('tblead','LEAD_Arquivado')`, `COL_LENGTH('tbleadcontato','LDC_NomeContato')`, `COL_LENGTH('tboportunidade','OPP_ProdutoId')` e `OBJECT_ID('tbproduto')` nÃ£o nulo |
| Reajuste sem data Ã© `null`, nÃ£o `0` | `rg -n 'dias \?\? 0' .\backend .\frontend` â€” nÃ£o pode haver `?? 0` no cÃ¡lculo; todo filtro de reajuste precisa testar `!== null` / `HasValue` |
| Faturamento zerado Ã© intencional | `rg -n 'FaturamentoLiberado' .\backend\Central_BackEnd` â€” com a constante `false`, `0` e lista vazia sÃ£o o comportamento correto, nÃ£o falha de cÃ¡lculo |
| SumÃ¡rio de sanidade | `tbcliente` 154 clientes, `tbcontrato` 185 contratos, 2021 licenÃ§as, 5 grupos, 6 clientes sem contrato, 101 contratos sem `DATA_PROXIMO_REAJUSTE` (referÃªncia de homolog, reconfirme) |

