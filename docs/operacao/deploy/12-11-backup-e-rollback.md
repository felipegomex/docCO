---
id: sec.12-11-backup-e-rollback
titulo: "11. Backup e rollback"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-28"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 1800
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 11. Backup e rollback

### Backup IIS

O parâmetro opcional `-Backup` tem padrão `$false`, mas o pipeline obrigatório usa `$true`. Antes da substituição, o script cria:

```text
C:\Users\JCASRV-SUP\Documents\Backup_IIS\<yyyyMMdd-HHmmss>\
├─ backend\
└─ frontend\
```

No backup do backend, `appsettings*.json` é excluído. O frontend é copiado integralmente. O backup não inclui banco de dados.

**Execução real de 2026-10-27 (HML + PROD).** Publicação do pacote sobre o HEAD `b52d7bc` (árvore de trabalho não commitada): backups IIS `20261027-233545` em HML (`.130`) e `20261027-233626` em PROD (`.254`); smoke final **5/5** nos dois ambientes. Em PROD o login respondeu **500** após o publish até aplicar os scripts de permissões — ver §13 e [`troubleshooting.md`](../troubleshooting.md) §20.

**Execução real de 2026-10-28 (HML + PROD).** Publicação do commit `f58cb6e` (branch `developer`, publicado em `origin`): backups IIS `20261028-192800` em HML (`.130`) e `20261028-192848` em PROD (`.254`); preflight de schema OK em `192.168.2.154` (HML `dbBUSINESS_HML`, PROD `dbCENTRAL_OPERACAO`); smoke **OK** nos dois ambientes (front 200, swagger 200, runtime-config OK, login 401 e webhook do WhatsApp 401 esperados). O `web.config` de cada servidor foi **preservado** no publish (porque tem `WhatsApp__*`), com cópias datadas `web.config.pre-deploy-20261028-192811` (HML) e `web.config.pre-deploy-20261028-192855` (PROD); em seguida o bloco de env vars do chat (`AnythingLLM__*`/`Gitea__*`/`KnowledgeSync__*`) foi gravado via SMB com a conta de serviço do deploy, com backup datado (`web.config.bak-20261028-192936` no HML, `web.config.bak-20261028-184627` no PROD). Detalhes em [`11-10-preservacao-de-configuracao.md`](11-10-preservacao-de-configuracao.md).

**Execução real de 2026-10-28 — `7ac8c61` e `a5e2df6` (HML + PROD).** Commit `7ac8c61` ("fix(chat): chama a API na origem da runtime-config e mostra o erro de carga do catalogo") publicado nos **dois** ambientes: backups IIS `20261028-201714` (HML `.130`) e `20261028-201735` (PROD `.254`); preflight de schema OK; smoke OK. **Incidente:** a condição `RegexPattern` que excluía `/api/*` do fallback SPA derrubou os **dois** sites do front — `500/50/13` no log do IIS (o módulo URL Rewrite não inicializa). A primeira restauração (`git show <commit>:frontend/public/web.config > arquivo` no **PowerShell**) gravou o arquivo em **UTF-16** e o IIS passou a responder **500.19**; a recuperação exigiu restauração em **bytes crus** (`cmd /c "git show ... > arquivo"`), que devolveu **200**. O commit `a5e2df6` ("fix(chat): reverte a regra de /api no fallback SPA do frontend") é **apenas a reversão do `web.config` no repo** — publicado **manualmente** nos dois servidores, porque o pacote local já carregava o arquivo correto após a restauração em bytes crus. Estado final verificado: front **200** e `runtime-config.json` **200** nos dois ambientes, smokes OK; `GET /api/chat/agentes` no PROD → **200** com **5** agentes para operador comum e **6** para admin.

**Execução real de 2026-10-30 (PROD, `6d2bbc1`).** Publicação do commit `6d2bbc1` (branch `developer`, push OK) **somente em PROD** (`.254`), reaproveitando o pacote HML sem rebuild: preflight de schema OK em `192.168.2.154` / `dbCENTRAL_OPERACAO`; backup IIS em `\\192.168.2.254\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261030-172340`; `web.config` com `WhatsApp__*` **preservado** (cópia `web.config.pre-deploy-20261030-172349`); smoke **5/5** (frontend 200, swagger 200, runtime-config com a API do próprio `.254:1009`, login 401 e webhook 401). HML (`.130`) **pendente**; o banco de PROD é `dbCENTRAL_OPERACAO` (a tabela de ambientes citava `dbBUSINESS` e foi corrigida).

### Rollback dos sites

Rollback é por ambiente — use o servidor e as pastas do `-Ambiente` publicado (ver seção 4). Exemplo para HML:

1. Identifique o timestamp e confirme que backend/frontend existem no backup.
2. Coloque o backend em manutenção pelo procedimento aprovado, preservando `app_offline.htm` se já existir.
3. Restaure os sites a partir do backup:

```powershell
$backup = '\\192.168.2.130\c$\Users\JCASRV-SUP\Documents\Backup_IIS\<timestamp>'

robocopy "$backup\backend" '\\192.168.2.130\c$\inetpub\wwwroot\Suporte_Back' /MIR /XF appsettings*.json app_offline.htm /R:2 /W:1
robocopy "$backup\frontend" '\\192.168.2.130\c$\inetpub\wwwroot\Suporte_Front' /MIR /R:2 /W:1
```

(Para PROD, troque servidor e pastas pelos do mapa `CentralOperacao\Back` / `CentralOperacao\FRONT`.)

4. Verifique que `appsettings*.json` e `runtime-config.json` atuais do servidor permaneceram intactos e que `Central_BackEnd.dll` e `Central_BackEnd.runtimeconfig.json` foram restaurados.
5. **Só depois do mirror íntegro**, remova o `app_offline.htm` remanescente; caso contrário, o rollback continuará offline:

```powershell
$offline = '\\192.168.2.130\c$\inetpub\wwwroot\Suporte_Back\app_offline.htm'
if (Test-Path -LiteralPath $offline) { Remove-Item -LiteralPath $offline -Force }
```

6. Recicle `Suporte_Back` se necessário.
7. Rode smoke e QA manual.

Se o backend novo depender de schema novo, restaurar apenas os sites pode causar incompatibilidade. Nesse caso, restaure também o banco com o plano do DBA ou aplique uma correção somente para frente; não improvise rollback de schema.

⚠️ **Rollback do módulo Acessos Empresas:** o `Down` da migration `20260928212856_AcessosEmpresas` **derruba as 7 tabelas e as 3 colunas `AAC_*`**, o que significa perder todos os cadastros de empresa/acesso e a trilha de auditoria do módulo. Um rollback de código que reverta o binário para a versão antiga (Google Sheets) deixa as tabelas órfãs — e elas contêm senha. **Não** reverta o schema automaticamente: preserve as tabelas e ajuste o código. Se for realmente necessário remover, recoloque primeiro um backup revisado com o DBA.

### Recuperação de queda total do front (`500.50.13` / `500.19`)

Sintoma: **todas** as requisições do site do front respondem **500** (`500/50/13`
no log do IIS) ou **500.19** (configuração ilegível).

1. **Não** edite o `web.config` do servidor com o redirecionamento `>` do
   PowerShell: ele grava **UTF-16** e produz **500.19**.
2. Restaure o arquivo do commit anterior **em bytes crus**:
   `cmd /c "git show <commit>:frontend/public/web.config > arquivo"` (o `cmd`
   preserva UTF-8 sem BOM) ou faça **cópia binária** de uma cópia boa.
3. Copie o arquivo para a raiz do site do front
   (`\\<servidor>\c$\inetpub\wwwroot\<site>\web.config`) e recycle do app pool.
4. Valide com `scripts/smoke.ps1 -Servidor <ip>`: front **200** e
   `runtime-config.json` **200** (o **401** do login fictício é o resultado
   esperado).
5. Só depois investigue a regra do URL Rewrite em `frontend/public/web.config`
   (repo) e publique pelo pipeline normal — **PROD e HML** ao mesmo tempo.

### Rollback de versão

Tags são a referência imutável. Use uma worktree/branch limpa baseada na tag, rode o pipeline completo e só promova se passar. Não faça deploy de uma árvore suja nem altere/mova uma tag já publicada.

### Banco próprio `dbCENTRAL_OPERACAO`

Desde 2026-10-14 o Central usa banco próprio, restaurado por `BACKUP COPY_ONLY` de `dbBUSINESS` (zero escrita no legado). Desde 2026-10-14 o Central usa banco próprio, restaurado por `BACKUP COPY_ONLY` de `dbBUSINESS` (zero escrita no legado). Regras: connection string de produção só muda com sync + verificação estáveis; migrations EF das tabelas do Central miram o banco novo, nunca `dbBUSINESS`.

