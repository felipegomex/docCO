---
id: sec.04-3-pre-requisitos
titulo: "3. Pré-requisitos"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - scripts/deploy
  - scripts/aplicar-whatsapp-sql.ps1
testes:
  - docs/
tokens_estimados: 618
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 3. Pré-requisitos

### Estação de trabalho

- Windows com PowerShell 5.1 ou superior.
- Git configurado e acesso ao remoto.
- Node.js 20, conforme o workflow do projeto, e npm.
- SDK .NET 8.
- Dependências frontend instaladas:

```powershell
Set-Location .\frontend
npm ci
Set-Location ..
```

- Dependências backend restauradas:

```powershell
Set-Location .\backend\Central_BackEnd
dotnet restore
Set-Location ..\..
```

- Acesso aos feeds de npm/NuGet e à rede interna.
- Para migrations: `sqlcmd`, SSMS ou ferramenta DBA aprovada e credencial obtida com segurança.
- SQL do commit aplicado **antes** do publish — inclui [`WhatsApp_Fase9_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase9_Idempotente.sql) (Fase 8, tabela `tbwhatsapp_mensagem_reacao`). O `preflight-schema.ps1` é **bloqueante** e aborta o deploy sem a tabela e o índice único. **Já aplicado em 2026-10-30** em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML` (`192.168.2.154`) — nesses dois bancos o preflight deixa de bloquear; para aplicar/conferir em outro banco use [`scripts/aplicar-whatsapp-sql.ps1`](../../../scripts/aplicar-whatsapp-sql.ps1) (`-ConfirmarBanco <banco>`, `-SomenteConferir`; credencial por `WHATSAPP_SQL_*` ou `-IntegratedSecurity`; banco sem `tbwhatsapp_mensagem` apenas imprime "ignorado").

### Servidor IIS

- ASP.NET Core Hosting Bundle compatível com .NET 8.
- IIS, ASP.NET Core Module e URL Rewrite para o fallback SPA.
- Bindings nas portas 1009 e 1010.
- Permissão de leitura/escrita para a conta operacional no compartilhamento administrativo `c$`.
- Pool `Suporte_Back` e configuração efetiva de ambiente/JWT/banco.
- Para Database Explorer, variáveis `DB_EXPLORER_*` protegidas no Application Pool.
- Para Chat e Conhecimento, reverse proxy/ARR ou regra equivalente que encaminhe `/api/chat/*` e `/api/conhecimento/*` ao backend `1009`; essa configuração não está versionada no repositório. O serviço de sincronização da VM usa `POST /api/sync` e `GET /api/status` na porta **8090** (`jca-sync-trigger.service`, header `X-Central-Sync-Key`), restrita no firewall a `.254` e `.130`.
- Swagger acessível em `/swagger/v1/swagger.json` (retorna HTTP 200 + JSON com `openapi` e `paths` não vazio), porque o smoke agora valida a definição OpenAPI, não apenas o HTML.

### Gate de segurança

- Não versionar `appsettings*.json` reais, senhas, chaves JWT, cookies ou strings de conexão.
- Obter a credencial do servidor por `Get-Credential`; não informar `-SenhaRemota` como texto (o script aborta sem `-SenhaRemota` explícito). Para não redigitar a cada deploy, a senha pode ficar em arquivo **local e não versionado**: o `.gitignore` do repositório já reserva `deploy.local.ps1`, `deploy.run.ps1` e `deploy.local.bat` para isso — nunca versione esse arquivo.
- Fazer backup do banco antes de migration; o backup IIS não inclui banco de dados. Deploy em `PROD` exige `-Backup 1` (trava aborta antes de tocar no IIS).

