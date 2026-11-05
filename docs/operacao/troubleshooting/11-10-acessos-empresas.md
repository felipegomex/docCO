---
id: sec.11-10-acessos-empresas
titulo: "10. Acessos Empresas"
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
tokens_estimados: 3629
ficha: docs/modulos/acessos.md
---
> Ficha: [Acessos Empresas](../../modulos/acessos.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 10. Acessos Empresas

MÃ³dulo CRUD em 6 tabelas (`tbclienteacesso`, `tbacessordp`, `tbacessovpn`, `tbacessoanydesk`, `tbacessobanco`, `tbacessoactyonweb`) â€” a antiga 7Âª (`tbgrupoacesso`) foi **dropada** e os grupos agora vÃªm da legada `tbcliente_grupo`. **Substituiu** a leitura de uma planilha do Google Sheets: `GoogleSheetsService`, a seÃ§Ã£o `GoogleSheets` do `appsettings` e os modelos `Empresa*` foram removidos. Se vocÃª procura `SpreadsheetId`/`ApiKey`/`Range` no cÃ³digo, estÃ¡ em material antigo.

`GET /api/v1/acessos/estado-cifra` Ã© o primeiro ponto de diagnÃ³stico: devolve `implementacao`, `ehCifradoDeVerdade` e `eventosAuditoriaPerdidos`.

### 10.1 Tabelas ausentes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `Invalid object name 'tbclienteacesso'` (ou `tb*acesso*`) | migration `20260928212856_AcessosEmpresas` nÃ£o aplicada no banco alvo | aplicar `Migrations/Sql/AcessosEmpresas_Idempotente.sql`; confirmar `__EFMigrationsHistory` |
| `Invalid object name 'tbgrupoacesso'` ou `Invalid foreign key constraint ... tbgrupoacesso` | script `AcessosGrupoClienteErp` nÃ£o aplicado (a tabela deve **nÃ£o existir** mais) | aplicar `Migrations/Sql/AcessosGrupoClienteErp_Idempotente.sql`; ver Â§10.1.1 |
| `Cannot insert duplicate key` em `IX_tbclienteacesso_CAO_ClienteId` | duas unidades apontando a mesma empresa do ERP | o Ã­ndice Ã© Ãºnico filtrado de propÃ³sito; use a unidade existente (`jaExistia: true`) |
| `Invalid column name 'AAC_ClienteAcessoId'` | as 3 colunas de auditoria faltam | o mesmo script; elas sÃ£o `NULL`-able, entÃ£o a tabela pode existir sem elas |
| `The index ... dependent on column ... is disabled` ao criar o CNPJ | `CREATE INDEX` filtrado exige os `SET` de sessÃ£o do padrÃ£o | rode o script como estÃ¡ (ele emite `ANSI_NULLS`/`QUOTED_IDENTIFIER`/`ANSI_PADDING`/`ANSI_WARNINGS`/`ARITHABORT`/`CONCAT_NULL_YIELDS_NULL`); no SSMS, habilite as opÃ§Ãµes em *Query Options* |
| `Cannot create a UNIQUE INDEX ... filter` / erro 1934 | filtro incompatÃ­vel com as opÃ§Ãµes da sessÃ£o | idem; nÃ£o remova o filtro â€” ele Ã© o que permite vÃ¡rios clientes sem CNPJ |
| `CREATE TABLE` falha embora o SQL pareÃ§a correto | jÃ¡ aplicado em outra transaÃ§Ã£o / objeto homÃ´nimo | o script Ã© idempotente e guardado por `__EFMigrationsHistory`; **nÃ£o** rode duas vezes com o histÃ³rico limpo |
| InMemory nÃ£o cria o schema | o backend nÃ£o chama `Migrate()` e o InMemory nÃ£o valida DDL | suba com `Database:UseSqlServer=true` para exercitar o mÃ³dulo |

âš ï¸ **O banco de homologaÃ§Ã£o Ã‰ o de produÃ§Ã£o** (`192.168.2.154 / dbBUSINESS_HML`, `compatibility_level 100`). **NÃ£o existe staging.** Toda migration Ã© mudanÃ§a de schema em produÃ§Ã£o de primeira vez. Procedimento adotado na `AcessosEmpresas`: backup antes â†’ SQL revisado â†’ migration sÃ³ aditiva â†’ script idempotente. Backup registrado: `CentralConhecimento_pre_AcessosEmpresas.bak`, 21.856 MB, no diretÃ³rio padrÃ£o de backup do servidor.

âš ï¸ **`compat 100` tambÃ©m vale para este mÃ³dulo.** Sem `OPENJSON`, nunca filtre em `IQueryable` por `Contains` de coleÃ§Ã£o local. A busca de clientes usa `ToLower().Contains(termo)` (vira `LIKE`, permitido), mas qualquer novo filtro por lista precisa ser agregado/carregado e filtrado **em memÃ³ria**.

### 10.1.1 Grupos via `tbcliente_grupo` â€” script `AcessosGrupoClienteErp`

A migration `20260930132443_AcessosGrupoClienteErp` dropa `tbgrupoacesso`, zera `CAO_GrupoId` antes do drop, converte a coluna para `smallint` e adiciona `CAO_ClienteId` + Ã­ndice Ãºnico filtrado + 2 FKs (`tbcliente_grupo`, `tbcliente`). O caminho em produÃ§Ã£o Ã© o script idempotente [`Sql/AcessosGrupoClienteErp_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/AcessosGrupoClienteErp_Idempotente.sql) â€” **jÃ¡ aplicado em `dbBUSINESS_HML` em 2026-09-30**; a seÃ§Ã£o continua valendo para outros ambientes.

âš ï¸ **Janela de incompatibilidade â€” a ordem Ã© obrigatÃ³ria (outros ambientes):** rode o script **imediatamente antes** do publish da API nova. O cÃ³digo antigo grava ids da `tbgrupoacesso` em `CAO_GrupoId`; depois do repoint da FK para `tbcliente_grupo` esses ids passam a ser invÃ¡lidos. PrÃ©-requisito: `AcessosEmpresas_Idempotente.sql` jÃ¡ aplicado.

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `THROW 50011`â€“`50014` ao rodar o script | prÃ©-requisito ausente (tabela/colunas da `AcessosEmpresas` faltando) | rode primeiro `Migrations/Sql/AcessosEmpresas_Idempotente.sql` |
| `Msg 547` ao dropar a FK antiga | constraint jÃ¡ removida em execuÃ§Ã£o anterior | o script trata ramo por ramo (`IF OBJECT_ID(...) IS NOT NULL`); Ã© idempotente |
| `Msg 1934`/Ã­ndice desabilitado ao criar o Ã­ndice de `CAO_ClienteId` | `CREATE INDEX` filtrado sem os `SET` de sessÃ£o | o script emite os 6 `SET` obrigatÃ³rios antes; rode o arquivo como estÃ¡ |
| Grupos aparecem vazios no modal/lista apÃ³s o deploy | `tbcliente_grupo` sem linhas no banco alvo | legada fora das migrations: confirme `SELECT COUNT(*) FROM tbcliente_grupo` no ambiente |
| Unidade duplicada para a mesma empresa | linhas antigas sem `CAO_ClienteId` (legado manual) que sobraram antes da `20261002161552` | **hoje nÃ£o deveria existir**: o script de drop apagou as unidades sem vÃ­nculo e o Ã­ndice Ãºnico bloqueia nova duplicata; se restar alguma no banco alvo, vincule-a ao ERP ou elimine-a e rode `Sql/AcessosRemoveIdentidadeLocal_Idempotente.sql` |
| CÃ³digo antigo ainda no ar apÃ³s o script | inversÃ£o de ordem (script antes do cÃ³digo) | reavalie os dados de `CAO_GrupoId` com o DBA antes de seguir; o `UPDATE ... SET CAO_GrupoId = NULL` do script sÃ³ roda uma vez (ramo do drop) |

### 10.2 Senhas, cifra e revelaÃ§Ã£o

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| Log `ACESSOS: credenciais sendo gravadas em TEXTO PURO` | `CifradorEmBranco` estÃ¡ registrado no DI | o DI esperado registra `CifradorAesGcm`; se o log aparecer, confirme o registro em `Program.cs`. Linhas antigas sem `v1:` ainda podem estar em texto puro e exigem backfill |
| Tela mostra o aviso de credenciais em texto puro | `ehCifradoDeVerdade = false` | confirme `GET /acessos/estado-cifra`; o aviso some quando virar `true` |
| Backend nÃ£o sobe ao trocar para `CifradorAesGcm` | `Acessos:ChaveCifra` ausente, Ã© marcador de placeholder, nÃ£o Ã© base64 ou tem tamanho â‰  32 bytes | o construtor **lanÃ§a**; a chave Ã© base64 de **32 bytes**, gerada no servidor com `CifradorAesGcm.GerarChaveBase64()`, nunca no repositÃ³rio. Placeholder (`TROQUE_POR_UMA_CHAVE`, `DEV_CHAVE_NAO_USAR`, `CHANGEME`, `CHANGE_ME`) Ã© detectado **antes** do decode â€” se a mensagem for "nÃ£o Ã© um base64 vÃ¡lido" num valor que claramente tem `_`, vocÃª estÃ¡ com versÃ£o antiga do `Cifrador.cs` |
| Cripto ligada, mas senhas antigas continuam em texto puro | **backfill nÃ£o executado** | `Decifrar` cai no caminho de compatibilidade (valor sem prefixo `v1:`) e devolve o texto puro **silenciosamente**. RODE o backfill: sem ele a aparÃªncia de seguranÃ§a nÃ£o corresponde Ã  seguranÃ§a real |
| Senha revelada volta ilegÃ­vel / `CryptographicException` | chave diferente da usada na gravaÃ§Ã£o, ou backfill parcial | `Decifrar` Ã© retrocompatÃ­vel (valor sem `v1:` volta como texto), entÃ£o o sintoma Ã© sÃ³ nas linhas jÃ¡ cifradas; a chave precisa ser a mesma do backfill |
| BotÃ£o de olho ausente em AnyDesk/Banco | **sem endpoint de revelaÃ§Ã£o para esses tipos**, por decisÃ£o | nÃ£o Ã© bug. RDP e VPN sÃ£o os Ãºnicos com `GET /{tipo}/{id}/senha` |
| Senha nÃ£o vem no `GET` de lista/detalhe | por contrato: sÃ³ `temSenha: boolean` | use o endpoint de revelaÃ§Ã£o; ele audita a chamada |
| 404 em `/rdp/{id}/senha` | acesso inexistente | a tela trata com toast; o id vem da lista do cliente |

âš ï¸ **A cifra nova nÃ£o cobre `tboperador`.** `tboperador.SENHA` Ã© `varchar(10)` em texto claro e o comparador (`SegurancaHelper.SenhasIguais`) hasheia os dois lados: o valor gravado equivale Ã  senha, entÃ£o ler a linha jÃ¡ permite autenticar. Migrar o login Ã© projeto separado; nÃ£o confunda os dois problemas.

### 10.3 Auditoria em fila

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `eventosAuditoriaPerdidos` > 0 em `/acessos/estado-cifra` | fila cheia (`TryWrite` falhou) ou processo encerrado com eventos pendentes | investigue `LogError` "AUDITORIA: fila cheia"; o contador **zera ao reiniciar o processo**, entÃ£o anote antes de reciclar |
| `LogError` "Falha ao gravar lote de N evento(s)" | constraint, coluna truncada demais ou banco indisponÃ­vel | o lote inteiro Ã© perdido de propÃ³sito (a mutaÃ§Ã£o jÃ¡ foi confirmada); amutaÃ§Ã£o **nÃ£o** Ã© desfeita |
| `tbauditoriaacesso` cresce mas `tbauditoriaimplantacao` nÃ£o | o evento era `REVEAL_SENHA` (vai sÃ³ na primeira) | confirme `AAC_AcessoTipo` preenchido â€” Ã© o vÃ­nculo confiÃ¡vel, nÃ£o a coluna legada `Empresa` |
| Linha de auditoria sem `AAC_*` | linha anterior Ã  migration (log pela empresa) | esperado: as colunas sÃ£o `NULL`-able por opÃ§Ã£o; nÃ£o sÃ£o Ã³rfÃ£os a corrigir |
| `Processador de auditoria encerrou por falha inesperada` | exceÃ§Ã£o nÃ£o capturada no loop | `LogCritical`; o hosted service nÃ£o reinicia sozinho â€” reinicie o pool do IIS e investigue a causa |
| A revelaÃ§Ã£o responde mas a linha demora a aparecer | a gravaÃ§Ã£o Ã© assÃ­ncrona | comportamento esperado; espere o lote. Se nunca aparece, veja `eventosAuditoriaPerdidos` |

âš ï¸ **A fila Ã© singleton em memÃ³ria.** Com mais de uma instÃ¢ncia do backend, cada uma tem a sua: a trilha fica dividida e o contador Ã© local. O `validate.ps1`/`deploy.ps1` publicam uma instÃ¢ncia, mas isso precisa ser conferido se o ambiente mudar.

### 10.4 Gate, autorizaÃ§Ã£o e validaÃ§Ãµes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| Gate pede a senha de novo logo apÃ³s validar | o ticket de `EDITAR` Ã© de **uso Ãºnico** e dura 2 min: cada escrita ou revelaÃ§Ã£o consome o seu, entÃ£o um segundo PUT legÃ­timo na mesma empresa exige nova senha | Ã© o comportamento correto, nÃ£o cache expirado â€” o frontend consome o ticket no envio; reautentique para a prÃ³xima operaÃ§Ã£o |
| Gate pede a senha a cada troca de aba | o ticket de `VISUALIZAR` (2 min) expirou ou foi emitido para outra empresa | o ticket Ã© escopado por `ClienteAcessoId`: o emitido na unidade A nÃ£o abre a B |
| 403 `TicketRecusado` em operaÃ§Ã£o sensÃ­vel | ticket expirado (2 min, ou 5 min se emitido com `lembrar: true`), jÃ¡ consumido, de outro operador ou de outra empresa | emita outro com `POST /acessos/clientes/{id}/autorizar` |
| Modal de senha abre jÃ¡ preenchido | comportamento intencional: o gate guarda usuÃ¡rio + Ãºltima senha aceita **sÃ³ em memÃ³ria JS** (nada em disco; limpa no logout e ao fechar a aba) para prÃ©-preencher o modal | risco assumido e documentado no `SenhaGateService`: trocar praticidade por exposiÃ§Ã£o em XSS/estaÃ§Ã£o desbloqueada; para voltar a exigir digitaÃ§Ã£o sempre, remover `registrarCredenciais`/prÃ©-preenchimento no gate |
| Aba Visualizar do modal nÃ£o reflete ediÃ§Ã£o feita na aba Gerenciar | `credenciais()` era carregado uma vez na abertura; voltar de aba nÃ£o recarregava | **corrigido**: voltar Ã  aba Visualizar chama `irParaVisualizar()`, que recarrega (ticket `VISUALIZAR` Ã© reutilizÃ¡vel); se expirou, a mensagem orienta a confirmar de novo |
| 500 `{"mensagem":"Erro interno no servidor."}` no `PUT`/`DELETE /clientes/{id}` | consumo duplo do ticket: o filtro `RequerTicketAcesso` consumia o ticket `EDITAR` (uso Ãºnico) e o service revalidava via `ExigirTicket` â†’ `JaConsumido` virava `TicketRecusadoException` sem `catch` no action | **corrigido**: `AtualizarClienteAsync`/`ExcluirClienteAsync` nÃ£o revalidam (filtro Ã© a porta Ãºnica, como documentado no atributo) e os dois actions tratam `TicketRecusadoException` â†’ 403; regra: rota **com** clienteId valida no filtro, rota **sem** valida no service â€” nunca os dois |
| 429 ao validar | polÃ­tica `validacao`: 50/min (100 em Development) | aguarde a janela; nÃ£o faÃ§a loop automÃ¡tico |
| Bloqueio apÃ³s 5 falhas | `BruteForceGuard`: 15 min | aguarde; o estado Ã© em memÃ³ria e zera ao reiniciar |
| 400 "Informe o IP ou o servidor do acesso" | RDP exige um dos dois | preencha `ip` **ou** `servidor` |
| 400 "IP invalido" | `ValidadoresAcesso.IpValido` exige IPv4 em 4 octetos | hostnames vÃ£o em `servidor`, nÃ£o em `ip` |
| 400 "Tipo de VPN invalido" | valor fora de `FortClient`/`OpenVPN`/`WireGuard` | use um dos trÃªs literais |
| 400 "Ambiente invalido" | valor fora de `PROD`/`HML`/`DEV` | idem |
| 400 "Selecione uma empresa do ERP (CLIENTE_ID)..." / "Cliente do ERP nao encontrado" / "Cliente do ERP esta inativo." | `POST /clientes` **exige** `clienteId` (cadastro manual foi removido); a empresa precisa existir e estar ativa na `tbcliente` | selecione uma sugestÃ£o do typeahead (`GET /clientes/busca-erp`) â€” nÃ£o hÃ¡ payload alternativo |
| 400 "Ja existe um cliente ..." sem mensagem de grupo | Ã­ndice Ãºnico filtrado de `CAO_ClienteId` (uma unidade por empresa do ERP) | abra a unidade existente (o fluxo do modal jÃ¡ faz isso com `jaExistia: true`) |
| 400 "Grupo nao encontrado ou inativo" / "Cliente nao encontrado ou inativo" | referÃªncia mandada para registro inativo | reative ou escolha outro |
| Flags da lista nÃ£o batem com as abas do detalhe | sÃ³ se o backend nÃ£o rodou `RecalcularFlagsAsync` | recarregue a lista; se persistir, Ã© bug â€” as flags **nÃ£o** sÃ£o editÃ¡veis pelo cliente |
| `temSenha` sempre `false` | âš ï¸ bug do sombreamento de `SenhaCifrada` na base | **nÃ£o** mova `SenhaCifrada`/`TemSenha` para `AcessoBase`; rode `dotnet test` em `backend/Central_BackEnd.Tests` (`TemSenhaTests` trava esse comportamento) |
| `tbacessoactyonweb` com coluna `SenhaCifrada` sobrando | banco com a migration `20260929013916_AcessosEmpresas_DropColunasOrfasActyonWeb` **nÃ£o** aplicada | aplique a migration (ou [Sql/AcessosEmpresas_DropColunasOrfasActyonWeb.sql](../../../backend/Central_BackEnd/Migrations/Sql/AcessosEmpresas_DropColunasOrfasActyonWeb.sql)) e regenere o snapshot. **NÃ£o** remova `Usuario`: Ã© coluna legÃ­tima, mapeada por heranÃ§a de `AcessoBase` |

