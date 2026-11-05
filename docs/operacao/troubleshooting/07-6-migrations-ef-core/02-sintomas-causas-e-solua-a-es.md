---
id: 07-6-migrations-ef-core.02
titulo: "Sintomas, causas e soluÃ§Ãµes"
tipo: operacao
area: diagnostico
sistema: central
tags: [migration, efcore, sqlserver, erro]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Migrations
  - scripts
testes:
  - docs/
tokens_estimados: 2376
ficha: docs/modulos/database-explorer.md
---
> Parte de [6. Migrations EF Core](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Sintomas, causas e soluÃ§Ãµes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `THROW 50001`â€“`50005` ao rodar `ColunasPorFuncao_Backfill_Idempotente.sql` | prÃ©-requisito ausente: Fase 0 do `SetorParaFuncao` nÃ£o aplicou `CLK_FuncaoId`/`TRF_FuncaoId`/`PRJ_FuncaoId`/`CLK_StatusCard`/`CLK_SincronizaAgenda` | rode primeiro `Migrations/Sql/SetorParaFuncao_Idempotente.sql` e reaplique |
| **quadro das funÃ§Ãµes vazio (sem colunas) depois do publish do cÃ³digo de colunas compartilhadas** | o `Migrations/Sql/ColunasCompartilhadas_Idempotente.sql` **nÃ£o foi aplicado antes** do publish â€” o cÃ³digo novo sÃ³ serve coluna do **catÃ¡logo** (`CLK_FuncaoId` nulo), e o banco ainda tem sÃ³ as colunas por funÃ§Ã£o | aplicar o script **imediatamente** (Ã© idempotente) e conferir as 6 contagens: 7 colunas de catÃ¡logo, **0** cÃ³pias por funÃ§Ã£o, visÃµes gravadas e **0** cards apontando para cÃ³pia; se o script jÃ¡ tinha rodado, o problema Ã© outro (veja a linha seguinte) |
| cards que sumiram do quadro apÃ³s o publish das colunas compartilhadas | o script rodou **depois** do publish, ou rodou sem o par (coluna de funÃ§Ã£o â†” coluna de catÃ¡logo pela `CLK_Ordem`) | o script remapeia por `CLK_Ordem` e aborta com `THROW 50006` se algum par nÃ£o existir; reaplique com backup e confira a contagem "cards em cÃ³pia" (= 0) |
| `THROW 50006` ("coluna â€¦ sem par no catalogo") ao rodar o script de colunas compartilhadas | existe coluna de funÃ§Ã£o cuja `CLK_Ordem` nÃ£o tem par no catÃ¡logo (as duas sÃ©ries precisam ir de 1 a 7) | alinhar as ordens antes de rodar; o script se recusa a remapear card para coluna errada |
| quadro de uma funÃ§Ã£o vazio ou com colunas de outra funÃ§Ã£o logo apÃ³s o deploy da SeÃ§Ã£o 3 | backfill `ColunasPorFuncao` nÃ£o rodou **antes** do publish (ou rodou sÃ³ numa funÃ§Ã£o) â€” **modelo de 2026-10-02, superado** em 2026-10-20 (hoje vale o script de colunas compartilhadas, ver acima) | aplicar `Migrations/Sql/ColunasPorFuncao_Backfill_Idempotente.sql` no banco alvo (Ã© idempotente, pode reaplicar) e conferir as 4 verificaÃ§Ãµes do prÃ³prio script: 0 funÃ§Ãµes sem coluna, 0 cards Ã³rfÃ£os |
| `400 "Coluna invÃ¡lida para a funÃ§Ã£o da tarefa/projeto."` ao salvar ou mover card | a coluna **nÃ£o estÃ¡ visÃ­vel no quadro do setor** (a regra Ã© de visibilidade, nÃ£o de posse: a coluna Ã© do catÃ¡logo compartilhado) â€” a funÃ§Ã£o da tarefa vem do projeto ou, sem projeto, do criador | o **gestor da funÃ§Ã£o** reativa a coluna na visÃ£o do setor (`/kanban/configurar` ou o bloco "VisÃ£o por setor" de `/kanban/admin/colunas`); pelo caminho antigo, criar a coluna no grupo da funÃ§Ã£o nÃ£o resolve mais |
| `400 "A coluna de destino nÃ£o estÃ¡ visÃ­vel neste quadro."` ao arrastar no quadro | o setor ocultou a coluna de destino (visÃ£o em `tbcolunafuncao`) | o gestor da funÃ§Ã£o reativa a coluna na visÃ£o do setor |
| `400 "SÃ³ Ã© possÃ­vel reordenar colunas de uma Ãºnica funÃ§Ã£o por vez."` | **mensagem do modelo antigo** â€” a reordenaÃ§Ã£o agora Ã© **global** (o catÃ¡logo inteiro); se ela aparece, o cÃ³digo em produÃ§Ã£o ainda Ã© o anterior | conferir a versÃ£o publicada (`BUILD_INFO.GitCommit`) |
| `400 "NÃ£o Ã© possÃ­vel desativar: existem cards nesta coluna."` | **comportamento esperado**: coluna com cards nÃ£o pode sair do quadro | mover os cards para outra coluna antes de desativar |
| querer "ver tudo" na administraÃ§Ã£o de colunas e nÃ£o achar | o `GET /colunas-kanban` sem `todas=true` devolve o **catÃ¡logo ativo** (sem `funcaoId`), e com `funcaoId=N` devolve a **visÃ£o do setor** | a tela de administraÃ§Ã£o usa `?todas=true` para montar o catÃ¡logo; via API, use `?todas=true` |
| `dotnet ef database update` mexe em LocalDB | factory de design usa `(localdb)\MSSQLLocalDB` | passe explicitamente a conexÃ£o do alvo ou aplique SQL revisado; nunca use a factory sem confirmar o banco |
| coluna/FK/Ã­ndice ausente em homolog | migration nÃ£o aplicada | gerar script do commit atual, revisar, executar e verificar schema/histÃ³rico |
| migration marcada no histÃ³rico, mas DDL ausente | execuÃ§Ã£o parcial ou histÃ³rico inconsistente | conferir o schema real; nÃ£o confiar apenas na mensagem do EF |
| migration â€œfantasmaâ€ no histÃ³rico | ID sem migration correspondente | snapshot e arquivos atuais sÃ£o a referÃªncia; corrigir o histÃ³rico com DBA e plano aprovado |
| `dotnet ef` falha ao criar migration | CLI ausente/incompatÃ­vel ou model snapshot inconsistente | usar CLI EF 8, restaurar projeto e corrigir o primeiro erro do modelo |
| migration falhou em cascata | mais de um caminho `CASCADE` para a mesma relaÃ§Ã£o | revisar `DeleteBehavior` e SQL; SQL Server exige regra explÃ­cita compatÃ­vel |
| produÃ§Ã£o estÃ¡ com schema do cÃ³digo anterior | deploy nÃ£o aplica migration | interromper o fluxo funcional, aplicar migration antes do prÃ³ximo publish e repetir smoke |
| `Invalid object name 'tbfuncaoetapa'` ou `Invalid column name 'TRF_FuncaoId'`, `'PRJ_FuncaoId'`, `'CLK_FuncaoId'` em `/implantacao/kanban/board` | `SetorParaFuncao_Idempotente.sql` (Fase 0) nÃ£o aplicado no banco alvo | aplicar a Fase 0 **antes** de publicar o cÃ³digo novo; o script de consolidaÃ§Ã£o aborta sozinho se ela nÃ£o tiver rodado |
| `Invalid object name 'tbsetor'` / `'tbetapsetor'` / `'tbbloqueio'` ou `Invalid column name 'TRF_SetorId'`, `'PRJ_SetorId'`, `'TRF_EtapaTemplateId'`, `'CLK_SetorId'`, `'CLK_StatusCard'` em `/implantacao/kanban/board` | `KanbanGeral_Idempotente.sql` nÃ£o aplicado no banco alvo | aplicar `Migrations/Sql/KanbanGeral_Idempotente.sql` (tem `DROP` em `TRF_Bloqueada`/`TRF_MotivoBloqueio`: faÃ§a backup antes) e conferir `__EFMigrationsHistory` |
| cards de RH/Comercial somem de todo recorte por funÃ§Ã£o | esses dois setores viraram `FUNCAO_ID NULL` no mapa de conversÃ£o â€” **nÃ£o existem como funÃ§Ã£o**; e a pill "Todas" saiu do quadro em 2026-10-01 | nÃ£o Ã© bug de filtro: hoje eles nÃ£o aparecem em **lugar nenhum**. Se precisarem de funÃ§Ã£o, crie a linha em `tbfuncao` e faÃ§a o ajuste manual dos cards |
| querer "ver tudo" no Kanban Geral e nÃ£o achar a opÃ§Ã£o | a pill **"Todas" foi removida** por decisÃ£o (2026-10-01): o quadro sempre abre sob uma funÃ§Ã£o | usar as pills por funÃ§Ã£o; o endpoint ainda aceita `funcaoId` nulo (= Todas) via URL/API, mas a tela nÃ£o expÃµe mais |
| coluna "BLOQUEADO" reaparecendo no quadro | coluna continua existindo e ativa no banco alvo | o script desativa (`CLK_Ativa = 0`) e o frontend tambÃ©m filtra `id = 22`/nome com `BLOQUEAD`; reaplique o script ou ajuste o filtro |
| `403` em mover card, abrir/fechar bloqueio, mesmo com permissÃ£o aparente | a origem estÃ¡ em **outra funÃ§Ã£o** (ou sem funÃ§Ã£o) e o operador nÃ£o Ã© `GESTOR_MASTER` nem gestor daquela funÃ§Ã£o | conferir `tboperador.FUNCAO_ID`/`CARGO_ID`, `tbgestor` e `TRF_FuncaoId`/`PRJ_FuncaoId` do card; a regra estÃ¡ em `FuncaoAuthorizationService` |
| `Sem permissÃ£o para alterar itens de outra funÃ§Ã£o.` na tela sem justificativa | a UI reproduz a matriz no cliente e pode estar desatualizada | comparar `KanbanComponent.ehGestor`/`podeEditarCard` com `FuncaoAuthorizationService`; o backend Ã© a fonte da verdade |
| quadro com `?perfil=` vazio ou sem projetos (deep link legado) | o board nÃ£o filtra por perfil e o cruzamento por id falha/vazio; `?perfil=` sÃ³ chega pela **URL digitada** â€” desde 2026-10-20 nenhum item de menu o envia (o "Kanban ADM" saiu por duplicar `/kanban`) | o filtro cruza `board?tipo=tarefas` com `GET /implantacao/tarefas?perfilId=&perfilModo=incluir&limite=2000`; projetos **nÃ£o** tÃªm filtro por perfil e sÃ£o ocultados de propÃ³sito. Para sair do recorte, Ã© sÃ³ abrir `/kanban` puro |
| dias parado sempre `0` em gargalos | `BLQ_DataInclusao` da linha aberta estÃ¡ zerada/ausente ou o relÃ³gio do servidor estÃ¡ errado | o aging Ã© `DateTime.Now - BLQ_DataInclusao`; confira a data da linha aberta em `tbbloqueio` |
| motivo do bloqueio "desapareceu" ao reabrir | reabrir um item jÃ¡ bloqueado **atualiza o motivo** da linha aberta | comportamento atual de `BloqueioService.AbrirAsync`; o histÃ³rico sÃ³ existe entre ciclos fechar â†’ abrir |
| `PUT /implantacao/tarefas/{id}` fechou o bloqueio de um card | o request traz `bloqueada=false` (aÃ§Ãµes em lote do Kanban) | `TarefaService.AtualizarAsync` chama `FecharAsync` sempre que `bloqueada` nÃ£o vem `true`; o request nÃ£o distingue "ausente" de "`false`" |
| banco novo com tabela `tbcolunakanban` (14 caracteres) | o snapshot do EF e os `Designer` de `20260928182047`+ declaram esse nome grafado, enquanto o modelo (`ColunaKanban.cs`) e os scripts usam `tbcolunakanban` (17) | **NÃ£o** use `dotnet ef database update` para o Kanban Geral nem para a conversÃ£o Setor â†’ FunÃ§Ã£o: aplique os scripts de `Migrations/Sql`. Ver a armadilha em [04-ESTRUTURA-DADOS.md](../../../arquitetura/banco-dados.md) |
| `dotnet ef migrations add` gerando um `RenameTable` da tabela de colunas do Kanban | o snapshot diverge do banco real pelo nome grafado `tbcolunakanban` | nÃ£o aceite a migration gerada; alinhe o snapshot com o DBA antes de produzir DDL novo. Foi exatamente isso que levou o `Up` de `20260929221556_SetorParaFuncao` a ficar **vazio de propÃ³sito** |

