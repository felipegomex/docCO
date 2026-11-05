---
id: sec.05-4-ambientes-portas-e-chat-conhecimento-reverse-proxy
titulo: "4. Ambientes, portas e Chat/Conhecimento/reverse proxy"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 2050
ficha: docs/modulos/chat-conhecimento.md
---
> Ficha: [Chat e Conhecimento](../../modulos/chat-conhecimento.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 4. Ambientes, portas e Chat/Conhecimento/reverse proxy

| `-Ambiente` | Servidor IIS | Backend IIS | Frontend IIS | Banco principal |
|---|---|---|---|---|
| `HML` | `192.168.2.130` | `inetpub\wwwroot\Suporte_Back` (`:1009`) | `inetpub\wwwroot\Suporte_Front` (`:1010`) | `dbBUSINESS_HML` |
| `PROD` | `192.168.2.254` (IIS PROD; SQL Server continua em `192.168.2.154`) | `inetpub\wwwroot\CentralOperacao\Back` (`:1009`) | `inetpub\wwwroot\CentralOperacao\FRONT` (`:1010`) | `dbCENTRAL_OPERACAO` |

Sem `-Ambiente`, o script usa servidor/pastas explícitos com aviso (prefira sempre `-Ambiente HML|PROD`). `PROD` exige `-Backup 1` e confirmação (`-ConfirmarProd PROD` ou digitar `PROD`) — a trava aborta **antes** de qualquer build ou toque no IIS. O preflight de schema lê `PREFLIGHT_DB_*`; em PROD o banco é `dbCENTRAL_OPERACAO` e em HML `dbBUSINESS_HML` (SQL Server `192.168.2.154`). O mesmo pacote serve aos dois ambientes (cada servidor preserva seu `appsettings*.json` e `runtime-config.json`): empacote uma vez, publique em HML, smoke, publique em PROD sem rebuild, smoke.

**Pré-requisito da publicação:** a autenticação no `c$` de PROD depende da **senha vigente** do usuário administrativo do IIS — obtenha-a antes do publish e passe-a em `-SenhaRemota` (o script não guarda credencial padrão) — e limpe as sessões de rede residuais do servidor antes de publicar (`net use \\<servidor>\C$ /delete /y`). Com sessão residual, o `New-PSDrive -Credential` da fase 5 falha com "várias conexões ... usando mais de um nome de usuário". Diagnóstico por mensagem em [`09-TROUBLESHOOTING.md`](../troubleshooting.md).

| Cenário | Frontend | Backend | Banco principal |
|---|---|---|---|
| Development/InMemory | `http://localhost:4200` | `http://localhost:1009` | `CentralDev`, efêmero |
| Development/SQL Homolog | `http://localhost:4200` | `http://localhost:1009` | SQL Homolog |
| IIS HML | `http://192.168.2.130:1010` | `http://192.168.2.130:1009` | SQL do servidor (`dbBUSINESS_HML`) |
| IIS PROD | `http://192.168.2.254:1010` | `http://192.168.2.254:1009` | SQL do servidor (`dbCENTRAL_OPERACAO` no SQL Server `192.168.2.154`) |

### InMemory

Em `ASPNETCORE_ENVIRONMENT=Development`, `Program.cs` consulta `Database:UseSqlServer`:

- `false`: usa EF Core InMemory `CentralDev`; dados desaparecem ao encerrar;
- `true`: usa `ConnectionStrings:DefaultConnection`.

A baseline atual de `appsettings.Development.json` usa `true`, ou seja, SQL Homolog. Para um teste rápido em memória, altere a flag para `false` antes de iniciar o backend. O provider InMemory não valida o schema SQL real.

### SQL Homolog

Para SQL Homolog, a baseline referencia `192.168.2.154` e `dbBUSINESS_HML`. Garanta rota, autenticação e migrations desse banco. Não copie a string de conexão para console ou documentação; prefira `user-secrets` ou configuração local protegida.

> ⚠️ **`dbBUSINESS_HML` é o banco de produção.** Não existe ambiente de staging. Uma migration aplicada aqui é mudança de schema em produção de primeira vez: faça backup, revise o SQL com o responsável pelo banco, mantenha a migration **aditiva** e use o script idempotente de `Migrations/Sql`. O procedimento seguido na `20260928212856_AcessosEmpresas` está na seção 7.

### API

- Desenvolvimento: `http://localhost:1009/api/v1`
- IIS HML: `http://192.168.2.130:1009/api/v1`
- IIS PROD: `http://192.168.2.254:1009/api/v1` (é também a `apiBaseUrl` esperada no `runtime-config.json` do servidor IIS PROD (`192.168.2.254`); o pre-flight e o smoke conferem esse valor)

A origem do frontend deve estar na política CORS do backend. A lista atual está em [`appsettings.json`](../../../backend/Central_BackEnd/appsettings.json) (`Cors:AllowedOrigins`, lida pelo `Program.cs`) e já inclui as origens `:1010` dos dois servidores.

### Chat/Conhecimento e reverse proxy

As rotas do Chat/Conhecimento são **sem versão**: o chat é iframe para o AnythingLLM autenticado por `GET /api/chat/sso` e o Conhecimento está em `/api/conhecimento/*` (as rotas antigas — `/api/chat/agentes`, `/api/chat/perguntar`, `/api/chat/provedor` — foram removidas no `cb36ad3`, 2026-11-03). O Angular **não** as chama por caminho relativo à raiz: usa `ConfigService#apiRootUrl` (host puro, derivado do `apiBaseUrl` do `runtime-config.json`), porque a versão só se aplica a `/api/v1/*` — sem runtime-config o serviço mantém o caminho relativo. O `web.config` versionado do **front** **não** exclui `/api/*` do fallback da SPA (a regra que excluía foi **revertida** em `a5e2df6`, depois de derrubar os dois sites com `500.50.13`); o front responde **200 com o `index.html`** em `/api/*` e o que garante o funcionamento é a URL na origem configurada — basta o backend `:1009` responder. A regra de encaminhamento efetiva continua a ser inventariada fora do repositório.

O chat é iframe para o AnythingLLM autenticado por `GET /api/chat/sso` (`ChatSsoController`); o conteúdo é processado pelo AnythingLLM e pelos provedores configurados nele. O serviço de sync (`scripts/knowledge-sync/`) lê a branch principal do Gitea e atualiza os workspaces; o roteamento pasta→workspace vem de `_config/workspaces.yml`. Antes de publicar, configure de forma protegida os segredos (`AnythingLLM:ApiKey`; provedor do AnythingLLM; `Gitea:OAuth:ClientSecret`, `Gitea:ServiceToken` e `KnowledgeSync:Secret`) e as URLs/OAuth da VM. O SQL de chat (ChatConhecimento, ChatAgente, ChatProvedor e ChatConfiguracaoSistema) está **aplicado em HML e PROD desde 2026-10-30**; `tbconfiguracao_sistema` guarda a chave-mestra e o token do Gitea. O backend não executa migrations no startup nem o deploy as aplica; o gate é o preflight de schema. O sincronizador tenta criar workspaces ausentes no primeiro sync apenas se a API AnythingLLM instalada suportar essa operação; isso não substitui o mapeamento função→workspace no banco.

**Pré-requisito para rejeição:** atualize `_config/status-documento.yml` no repositório Gitea para permitir `rejeitado` antes de habilitar/usar esse fluxo. A rejeição agora cria PR de conta de serviço, que é aprovado e mesclado com o OAuth pessoal do revisor, mantendo `status: rejeitado` no caminho original.

O [guia `scripts/knowledge-sync/README.md`](../../../scripts/knowledge-sync/README.md) descreve o sync rodando na VM em `/opt/jca/sync/app` e o gatilho `POST /api/sync` + `GET /api/status` na porta **8090**, autenticados por `X-Central-Sync-Key`; o serviço é o `jca-sync-trigger.service` (systemd), com o mapa pasta→workspace em `_config/workspaces.yml` e manifesto fixo em `/opt/jca/sync/state/manifest.json`. A Central (`.254`/`.130`) chama o gatilho e consulta seu estado por `KnowledgeSync__TriggerUrl`/`KnowledgeSync__Secret`. Restrinja a porta no firewall da VM à Central (hoje ufw libera 8090/tcp para `.254` e `.130`) e armazene o segredo só na configuração protegida dos dois hosts.

Os scripts `Migrations/Sql/ChatConhecimento_Idempotente.sql`, `Migrations/Sql/ChatAgente_Idempotente.sql`, `Migrations/Sql/ChatProvedor_Idempotente.sql` e `Migrations/Sql/ChatConfiguracaoSistema_Idempotente.sql` estão no repositório e **foram aplicados em HML e PROD em 2026-10-30** (4 migrations de chat); o backend não executa migrations no startup nem o deploy as aplica. `scripts/aplicar-chat-sql.ps1` aplica os **4** SQLs e exige as **4** migrations. Revisores (`tbconhecimento_revisor_funcao`) continuam sem seed. OAuth do Gitea, segredos, sync/trigger, `.env` restrito e rota de rede da VM **já foram configurados**; o que permanecer pendente é registrado na operação (ex.: `Gitea__ServiceToken`, sem o qual a aprovação responde `503`).

