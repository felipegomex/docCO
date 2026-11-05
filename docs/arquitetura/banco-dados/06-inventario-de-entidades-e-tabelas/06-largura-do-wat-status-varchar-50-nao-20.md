---
id: 06-inventario-de-entidades-e-tabelas.06
titulo: "Largura do `WAT_Status` — `varchar(50)`, não 20"
tipo: referencia
area: dados
sistema: central
tags: [schema, tabelas, entidades, sqlserver]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
  - backend/Central_BackEnd/Migrations
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2488
ficha: docs/modulos/database-explorer.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

#### Largura do `WAT_Status` — `varchar(50)`, não 20

`tbwhatsapp_numero.WAT_Status` guarda o **status canônico** da sessão do gateway (`created`, `initializing`, `qr_ready`, `authenticating`, `ready`, `disconnected`, `action_required`, `failed`) — **não** o nome do evento. A coluna é **`varchar(50)`**, e o modelo (`Models/WhatsApp/Numero.cs`) declara `[MaxLength(50)]` para concordar com ela.

**Por que 50.** O primeiro deploy do script criou a coluna com **`varchar(20)`**. O handler de status de sessão gravava o **nome do evento** (`session.authenticated`, **21 caracteres**), o SQL Server recusava o INSERT/UPDATE com o **erro 8152** ("dados de cadeia ou binários seriam truncados"), a linha da fila ficava **presa em "processando"** e o **telefone do número nunca era gravado**. A causa **não** era a fila: era o **nome do evento** em coluna curta.

**Correção em três partes:**

1. o código grava **sempre um status canônico** (`WhatsAppWebhookService.StatusDaSessao`) — o nome do evento nunca vai para a coluna;
2. a coluna passa a `varchar(50)` e o modelo passa a `[MaxLength(50)]`;
3. o script ganhou um **bloco idempotente de ajuste de largura**, que alarga a coluna em base já instalada **sem recriar a tabela** (guardado por `sys.columns`/`sys.types`): o `ALTER` só acontece quando a coluna ainda é estreita, e reaplicar o script é no-op.

⚠️ **O script precisa ser reaplicado** quando houver mudança de largura: publicar o código novo **sem** reaplicar o script mantém o `varchar(20)` e o erro 8152 volta. Diagnóstico em [`operacao/troubleshooting.md`](../../../operacao/troubleshooting.md) §19.10.1 e procedimento em [`operacao/deploy.md`](../../../operacao/deploy.md) §7.

**Índices** (todos criados fora do `CREATE TABLE`, para que a reaplicação sobre tabela existente conserte índice faltante sem recriar a tabela):

| Tabela | Índice | Tipo |
|---|---|---|
| `tbwhatsapp_numero` | `UX_…_SessaoId`, `IX_…_Ativo` (legado, só onde a coluna `WAT_Ativo` ainda existe), **`IX_…_WAT_FuncaoId`** (FK do setor) | único + não únicos |
| `tbwhatsapp_contato` | `UX_…_ChatId`, `UX_…_TelefoneNormalizado` (**filtrado**, `WAT_TelefoneNormalizado IS NOT NULL`), `IX_…_Fundido`, `IX_…_EmpresaId` (novo ciclo) | únicos (um filtrado) + 2 não únicos |
| `tbwhatsapp_conversa` | `UX_…_NumeroId_ChatId`, `IX_…_AtendimentoId` | único + não único |
| `tbwhatsapp_atendimento` | `IX_…_Status`, `IX_…_ResponsavelId`, `IX_…_NumeroId`, `IX_…_UltimaMensagemEm`, `IX_…_EmInvestigacao` (Fase 06), **`UX_…_Protocolo` (único, Fases 0/1)** | os quatro filtros do Kanban + a marca de investigação + a busca exata por protocolo |
| `tbwhatsapp_mensagem` | `UX_…_ChatId_ExternalId` (**filtrado**, `WAT_ExternalId IS NOT NULL`), `IX_…_AtendimentoId_RecebidaEm`, `IX_…_ExternalId` | único filtrado + 2 não únicos |
| `tbwhatsapp_evento` | `IX_…_AtendimentoId_DataHora`, `IX_…_Tipo_DataHora`, `IX_…_OperadorId` | não únicos |
| `tbwhatsapp_webhook_fila` | `UX_…_Chave`, `IX_…_Estado_ProximoProcessamentoEm` | único + não único |
| `tbwhatsapp_parametro` | `UX_…_Chave` | único |
| `tbwhatsapp_resposta_rapida` | `UX_…_Atalho`, `IX_…_Ativo_Categoria` | único + não único |
| `tbwhatsapp_sla_config` | `IX_…_Ativo` | não único |
| `tbwhatsapp_sla_horario` | `IX_…_DiaSemana_Inicio` | não único |
| `tbwhatsapp_sla_historico` | `IX_…_DataInclusao` | não único |
| `tbwhatsapp_retencao_execucao` | `IX_…_InicioEm` | não único |
| `tbwhatsapp_etiqueta` | `UX_…_NomeAtiva` (**filtrado**, `WAT_Ativo = 1`) | único filtrado |
| `tbwhatsapp_atendimento_etiqueta` | `IX_…_Etiqueta` | não único (a PK cobre o atendimento) |

O índice único filtrado de `tbwhatsapp_mensagem` é o que impede a **reentrega** do gateway (entrega é *at-least-once*) de virar mensagem duplicada. O índice único de `tbwhatsapp_webhook_fila.WAT_Chave` é a chave de dedupe do webhook.

**Fases 0/1 do plano de evolução (2026-10-27) — schema novo.** [`WhatsApp_Fase8_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase8_Idempotente.sql) adiciona `WAT_Protocolo` (computada PERSISTED) + índice único, `WAT_FotoUrl`/`WAT_FotoAtualizadaEm`/`WAT_TelefoneResolvido` em `tbwhatsapp_contato`, faz o backfill de `WAT_Numero` removendo o JID e insere os dois parâmetros. **Aditivo e idempotente**; rode no banco do ambiente **antes** do publish do backend que lê as colunas novas. **Aplicado 2× em `dbBUSINESS_HML` e 2× em `dbCENTRAL_OPERACAO` em 2026-10-27** (segunda execução no-op), com backups `dbBUSINESS_HML_pre_fase8_whatsapp_20261027-232815.bak` e `dbCENTRAL_OPERACAO_pre_fase8_whatsapp_20261027-232853.bak`.

**Número vinculado a um setor (`tbfuncao`) — transição em dois scripts.** O cadastro deixou de ter texto livre (`WAT_Fila`) e passou a ter FK para o setor. A transição é dividida para não quebrar a janela entre SQL e publish:

1. [`WhatsApp_NumeroFuncao_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_NumeroFuncao_Idempotente.sql) — **aditivo, antes do publish**: adiciona `WAT_FuncaoId int NULL`, faz o **backfill** casando `WAT_Fila` com `tbfuncao.DESCRICAO` (`UPPER`/`TRIM`), aplica fallback ("Suporte" ativo ou a primeira função ativa; aborta com `THROW` se não houver função ativa) e cria a FK `FK_tbwhatsapp_numero_tbfuncao` + o índice `IX_tbwhatsapp_numero_WAT_FuncaoId`. **Não dropa `WAT_Fila`.**
2. [`WhatsApp_NumeroFuncao_Consolidar_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_NumeroFuncao_Consolidar_Idempotente.sql) — **somente depois do backend novo publicado**: refaz o fallback para linhas criadas pelo código antigo, altera `WAT_FuncaoId` para **NOT NULL** e **dropa `WAT_Fila`**. É **irreversível** (o drop não tem volta) e aborta se o script aditivo não tiver rodado.

Os dois são idempotentes, não carimbam `__EFMigrationsHistory` e não criam migration EF. O script base `WhatsApp_Idempotente.sql` ainda cria `WAT_Fila` em banco novo — por isso os dois scripts acima também precisam rodar em base nova, na mesma ordem. O `preflight-schema.ps1` exige `WAT_FuncaoId` e a FK antes do publish.

**Regra de ouro do banco.** Os scripts são **aditivos**, com **quatro** exceções conscientes: (a) o **bloco idempotente de ajuste de largura** de `tbwhatsapp_numero.WAT_Status` (`ALTER COLUMN … varchar(50)`), que só age quando a coluna ainda é estreita; (b) a Fase 5 **relaxa** `tbwhatsapp_evento.WAT_AtendimentoId` para aceitar nulo (`ALTER COLUMN … NULL`); (c) a **F6** remove e recria a `CHECK` `CK_tbwhatsapp_atendimento_Status` (`DROP CONSTRAINT`/`ADD CONSTRAINT`) para aceitar **0..6**, sem tocar em dados nem em coluna; e (d) a **consolidação do setor do número** (`WhatsApp_NumeroFuncao_Consolidar_Idempotente.sql`) marca `WAT_FuncaoId` como **NOT NULL** e **dropa `WAT_Fila`** — por isso ela roda **depois** do deploy, nunca antes. Fora disso, só `CREATE TABLE`, `CREATE INDEX` e `ALTER TABLE … ADD CONSTRAINT`: nenhum `DROP` de tabela/coluna nem `DELETE`. Rodar os scripts **aditivos** com o código antigo no ar não quebra nada (as tabelas/colunas novas ficam sem uso) e reaplicar é no-op; a consolidação do setor (d) é a exceção e só roda **depois** do deploy. Ainda assim, **o banco de homologação é o de produção** (`dbBUSINESS_HML`, `compatibility_level 100`, sem staging): faça backup antes, rode os scripts **antes** do publish do backend com `/api/v1/whatsapp` (a consolidação, depois) e confira a conferência final que cada script imprime. O `deploy.ps1` **não** aplica migrations.

**Estado em 2026-10-21:** o script base `WhatsApp_Idempotente.sql` foi revisado e **aplicado** em `dbBUSINESS_HML` (SQL Server `192.168.2.154`) — 7 tabelas `tbwhatsapp_*`, 11 FKs, 7 `CHECK`, 89 colunas; **reaplicação conferida = no-op**; **sem backup**, por decisão explícita do dono do projeto. Aplicado **em lotes** (o `SqlClient` não entende o separador `GO`); outros bancos do servidor (`dbCENTRAL_OPERACAO`, do `.254`) ainda **não** têm as tabelas. Os scripts das fases 0, 1, 3, 4, 5, 6, 7 e 8 são **adicionais e idempotentes**: rode-os no banco do ambiente antes do publish do código que usa as telas novas. A Fase 06 (novo ciclo) entra por `WhatsApp_Fase7_Idempotente.sql`; sem ele, as rotas de etiquetas/investigação/observação falham por objeto/coluna inexistente. **A F6 tem o script [`WhatsApp_Visto_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Visto_Idempotente.sql)**: ele remove e recria a `CHECK` `CK_tbwhatsapp_atendimento_Status` para **0..6**, sem coluna nova; **aplicado em HML e PROD em 2026-10-26**, com backup `COPY_ONLY` antes (`dbBUSINESS_HML_pre_visto_20261026-192400.bak` e `dbCENTRAL_OPERACAO_pre_visto_20261026-192449.bak`, ambos com `RESTORE VERIFYONLY`). **A Fase 07 não tem script**: o Kanban Service Desk só lê o schema existente. **A Fase 09 também não tem script nem alteração de schema**: tempo real e digitação usam `tbwhatsapp_atendimento`, `tbwhatsapp_conversa` e `tbwhatsapp_numero` já existentes. O módulo continua **desabilitado** no `appsettings.json` versionado (`WhatsApp:Habilitado = false`) e é ligado por variável de ambiente no servidor (`WhatsApp__Habilitado=true`); a chave da API e o segredo do webhook vêm de `user-secrets` em dev e de variável de ambiente no servidor. **Piloto em operação na homologação** (`.130`): número pareado gravado em `tbwhatsapp_numero` (sessão `central-wa-7`, status `ready`, telefone `558592644863`), mensagens reais recebidas e atendimentos abertos no Kanban. Cenários do módulo em [`referencia/qa-historias.md`](../../../referencia/qa-historias.md) §12; instalação do gateway em [`operacao/instalacao-openwa.md`](../../../operacao/instalacao-openwa/INDEX.md).

