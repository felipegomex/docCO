---
id: 05-4-servicos-principais.16
titulo: "Compatibilidade com o banco legado (restrição permanente)"
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
tokens_estimados: 727
ficha: docs/modulos/database-explorer.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Compatibilidade com o banco legado (restrição permanente)

O banco legado roda em `compatibility_level 100` (SQL Server 2008), onde **`OPENJSON` não existe**. Quando o provider **não** conhece o nível, o EF Core 8 emite `OPENJSON(@p)` para `lista.Contains(x.Id)`, o que produz **"Sintaxe incorreta próxima a '$'"**.

- **Nível declarado no provider (correção do SLA):** `Program.cs` registra o SQL Server com `UseCompatibilityLevel(100)` nos dois ramos (`NivelCompatibilidadeSqlServer`). Com o nível informado, o EF 8 volta a traduzir `Contains` de lista como `IN (...)` e deixa de emitir `OPENJSON`. Antes disso, a consulta do SLA falhava com `SqlException` e o `try/catch` de `CalcularSlaAsync` escondia o erro — o SLA não aparecia na tela, sem 500. A declaração não substitui o padrão explícito abaixo: `EF.Constant` continua sendo usado onde já existe e é a proteção legível para quem lê a consulta.
- **Não use** `Contains` de coleção local em nenhuma query, nem `Intersect`/`Except` sobre coleções LINQ.
- **Padrão obrigatório:** carregar/agregar a tabela pequena inteira e filtrar em memória. `ComercialService.ListarLeadsAsync` faz isso para as oportunidades (agrega `tboportunidade` inteira e indexa por `LeadId`) e para os contatos (carrega `tbleadcontato` e agrupa em memória) — antes o `leadIds.Contains(...)` derrubava a tela de leads.
- Alternativa quando o filtro é por chave única: subquery correlacionada (`EXISTS`) ou `IN` com subquery, como em `ProjetoService.ListarClientesAsync`.
- **Lista de ids calculada em memória:** use `EF.Constant(ids).Contains(...)` para fixar `IN` literal **na própria consulta**. É o que a Fase 07 faz no recorte do SLA (`AplicarIds`) e nas etiquetas do card (`QueryEtiquetas`). Com o nível 100 declarado, o EF já traduz `Contains` como `IN`; o `EF.Constant` continua sendo a forma explícita e independente de configuração.
- **Aplicar/remover etiquetas no atendimento** (`AtendimentoService.DefinirEtiquetasAsync`) segue a mesma regra nos **três** pontos que consultam o catálogo — validação do conjunto pedido, nomes das adicionadas e nomes das removidas — via o helper interno `QueryEtiquetasPorIds(db, ids)` (`EF.Constant`). Antes, o `Contains` sobre a lista gerava `OPENJSON` e o `PUT /atendimentos/{id}/etiquetas` caía em `SqlException 102` no banco legado.
- O mesmo padrão já é adotado em `DashboardService`, `TarefaService` (2 pontos ao validar chamados legados), `ColunaKanbanService` e `AgendaService`; os comentários no código registram a razão.
- O provider InMemory **não** reproduz o erro. Teste contra SQL Server real.

