---
id: sec.17-contagens-e-fatos-confirmados-no-baseline
titulo: "Contagens e fatos confirmados no baseline"
tipo: referencia
area: dados
sistema: central
tags: [schema, banco, tabelas]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1060
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Contagens e fatos confirmados no baseline

- 78 `DbSet`; 48 entidades incluídas nas migrações, 6 espelhos legados somente leitura, **20 conjuntos do módulo de WhatsApp** e **4 do RBAC** (tabelas criadas por script próprio, sem classe de migração).
- 36 classes de migração presentes; a mais recente é `20261030170000_ConfiguracaoSistema`. As migrations de chat (`20261021003310_ChatConhecimento`, `20261022165354_ChatAgentes`, `20261029213303_ChatProvedorConfig`, `20261030170000_ConfiguracaoSistema`) e os respectivos scripts idempotentes foram **aplicados em 2026-10-30** em `dbBUSINESS_HML` e `dbCENTRAL_OPERACAO`; a presença dos arquivos não comprova execução — confira `__EFMigrationsHistory` no ambiente. O módulo de WhatsApp **não tem migration EF**: o DDL está em `Migrations/Sql/WhatsApp_Idempotente.sql`, **aplicado em `dbBUSINESS_HML` em 2026-10-21** (idempotente; outros bancos do servidor ainda precisam dele). **A Fase 09 (tempo real e digitação) não acrescentou migration nem script**: o schema permanece com as 21 tabelas `tbwhatsapp_*` daquele momento. A **Fase 8 (reações)** acrescenta a 22ª tabela (`tbwhatsapp_mensagem_reacao`) por [`WhatsApp_Fase9_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase9_Idempotente.sql), **aplicado em 2026-10-30** em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML` (ambos em `192.168.2.154`) via [`scripts/aplicar-whatsapp-sql.ps1`](../../../scripts/aplicar-whatsapp-sql.ps1), com tabela, coluna `WAT_Emoji` e índice único conferidos — o preflight de deploy não bloqueia mais nesses dois bancos.
- `tbfuncao` é o **eixo do Kanban Geral** e tem **5** linhas na base real (2026-10-21): 1 `Analista de Sistemas`/`Implantador`, 2 `Suporte`/`Atendimento`, 3 `Programador`/`Desenvolvimento`, 4 `Executivo(a) Comercial` e 5 `CIAA`. `FuncaoFinanceiro_Idempotente.sql` não tem migration EF: cadastro de função é só dado. ⚠️ A função é a **fonte da identidade do eixo** (recorte do quadro e dono do card), mas **não** é fonte de **super-usuário**: quem lê/edita tudo é o cargo `GESTOR_MASTER` (ou a ponte `SE_ADMIN`/`PERFIL_ID = 'A'`) e quem gerencia funções está em `tbgestor` (ver [`FuncaoAuthorizationService`](../../../backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs)). O **cargo RBAC** é outro eixo (`tboperador.CARGO_ID` → `tbcargo`), com permissão lida de `tbperm_cargo`/`tbperm_operador`.
- `tbrefreshtoken` e as duas tabelas de auditoria são persistidas no `AppDbContext`.
- `tbcliente`, `tbcliente_grupo`, `tbchamado`, `tbfuncionario`, `tbcontrato` e `tbproduto` são fontes legadas de leitura; os dados do Explorer não são tabelas do `AppDbContext`.
- A listagem de clientes de projeto usa `tbcliente`; não existe mais `IMPL_Cliente` nem entidade `Cliente` no modelo. O módulo Acessos Empresas tem sua **própria** entidade `ClienteAcesso`/`tbclienteacesso`, que **não** é a mesma coisa: as duas convivem.
- Nomes de tabela seguem o padrão `tb*`; nomes de coluna, PKs e FKs seguem as regras descritas na seção de nomenclatura.
- **O `AppDbContextModelSnapshot` diverge do banco** e `dotnet ef migrations add` **não deve** ser rodado neste repositório até a correção. Já foi tentado em 2026-10-11 e gerou um `DROP COLUMN` em `tboperador`. Ver [a seção dedicada](#o-snapshot-do-ef--estado-da-sincronia-e-por-que-dotnet-ef-migrations-add-segue-perigoso-aqui).
- **RBAC aplicado em `dbBUSINESS_HML` em 2026-10-21**: as 4 tabelas (`tbcargo`, `tbpermissao_recurso`, `tbperm_cargo`, `tbperm_operador`) e `tboperador.CARGO_ID` existem; há 10 cargos, **8** operadores com `GESTOR_MASTER`, **1** com `TECH_LEAD_IMPLANTACAO` e **32** sem cargo (Visualizador). As 4 tabelas **não têm classe de migração EF** e não estão no snapshot; os scripts são `Permissoes_Idempotente.sql`, `Permissoes_CargaInicial_Idempotente.sql` e `Permissoes_SeedOperadores_Idempotente.sql`. Em **2026-10-27**, `Permissoes_Idempotente.sql` passou a criar `CARGO_Nivel`/`CARGO_FuncaoId` e o cargo `TECH_LEAD_ADM` (**11** cargos); `Permissoes_Matriz_Idempotente.sql` acrescentou a permissão de `aba.sistema.permissoes` aos Tech Leads e a linha por função de `aba.ciaa.ideias` (`PEC_FuncaoId = 5`). Reaplique os dois scripts no ambiente — o primeiro **antes** do segundo.
- **Não há mais nada do Google Sheets**: `Services/GoogleSheetsService.cs`, `IGoogleSheetsService`, a seção `GoogleSheets` do `appsettings` e os modelos `EmpresaAcesso`/`EmpresaResumo`/`EmpresaDetalheResponse` foram removidos. `PasswordValidationService`, `BruteForceGuard` e `AuditoriaAcesso`/`tbauditoriaacesso` foram **mantidos** e reusados pelo módulo.
