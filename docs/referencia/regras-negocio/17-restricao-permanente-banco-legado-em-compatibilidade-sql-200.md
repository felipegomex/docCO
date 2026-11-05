---
id: regras-negocio.17
titulo: "Restrição permanente: banco legado em compatibilidade SQL 2008"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 527
ficha: docs/modulos/database-explorer.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Database Explorer](../../modulos/database-explorer.md).

## Restrição permanente: banco legado em compatibilidade SQL 2008

**Situação: implementada como regra do projeto; vale para todos os serviços, não só para o Dashboard.**

- O banco legado `dbBUSINESS_HML` roda em `compatibility_level 100` (SQL Server 2008), onde **`OPENJSON` não existe**. O mesmo vale para `dbCENTRAL_OPERACAO` e `dbBUSINESS`.
- **O `Program.cs` declara o nível no provider:** os dois ramos SQL Server usam `sql => sql.UseCompatibilityLevel(NivelCompatibilidadeSqlServer)` (`NivelCompatibilidadeSqlServer = 100`). Com o nível informado, o EF Core 8 traduz `Contains` de lista como `IN (...)` e **não** emite `OPENJSON`. Sem essa declaração, qualquer um destes formatos estoura a query com **"Sintaxe incorreta próxima a '$'"**:
  - `leadIds.Contains(o.LeadId)` e listas equivalentes de agregação;
  - `Intersect`/`Except` sobre coleções locais;
  - qualquer filtro de "id está na lista" com a lista carregada pelo LINQ.
- **Padrão obrigatório:** agregar ou carregar a tabela pequena **inteira** e filtrar/intersectar **em memória**. É o que já fazem `DashboardService`, `TarefaService`, `ColunaKanbanService` e `AgendaService`.
- Alternativa aceitável quando o filtro é por chave única: subquery correlacionada (`EXISTS`) ou `IN` com subquery, como em `ProjetoService`.
- **Lista de ids já calculada em memória:** `EF.Constant(ids)` continua sendo usado no código existente (`AtendimentoService.AplicarIds` no recorte do SLA e `QueryEtiquetas` da Fase 07) como proteção **explícita e legível**, mesmo com o nível declarado no provider.
- O provider InMemory **não** reproduz esse erro: uma tela funcionar em memória não prova que a consulta roda no banco real. Todo serviço que lê as legadas precisa ser exercitado contra SQL Server antes de ser considerado válido.

