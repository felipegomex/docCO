# Checklist HML — Chat e Conhecimento

> **Status (2026-11-05):** esta página é um **registro histórico** do aceite em HML/PROD. O catálogo de agentes e o painel de provedor foram **removidos do produto** no `cb36ad3` (2026-11-03); o chat vigente é iframe + SSO. O SQL (4 scripts) foi aplicado em HML e PROD em **2026-10-30**; a publicação nos dois está OK. Não execute este checklist de novo; para operação corrente use `operacao/troubleshooting/12-11-chat-e-conhecimento.md`.

**Objetivo:** registrar o que foi aplicado, publicado e verificado no HML para o chat RAG e a tela de conhecimento (schema e publicação em 2026-10-28; sync/trigger em 2026-10-30) e o que ainda depende de pessoas.
**Escopo:** banco, segredos, validação e publicação. Nenhum valor de segredo entra neste arquivo.

> **Bancos (2026-10-28):** HML = `192.168.2.154/dbBUSINESS_HML`; PROD = `192.168.2.154/dbCENTRAL_OPERACAO`.
> HML **já estava migrado**, mas faltava o agente `ciaa`; o SQL idempotente foi
> reaplicado em 2026-10-28 com `scripts/aplicar-chat-sql.ps1`
> (`-ConfirmarBanco dbBUSINESS_HML`). PROD foi aplicado no mesmo dia no
> `dbCENTRAL_OPERACAO`. Os dois têm agora **6 agentes**, `AGENTE`,
> `tbchat_workspace_funcao` e 2 migrations. O aviso anterior (“não executar em
> produção”) foi substituído por essa decisão.

---

## a) SQL — ordem obrigatória

Aplicar com `sqlcmd`/`SqlClient`, no banco confirmado:

1. `backend/Central_BackEnd/Migrations/Sql/ChatConhecimento_Idempotente.sql`
2. `backend/Central_BackEnd/Migrations/Sql/ChatAgente_Idempotente.sql`
3. `backend/Central_BackEnd/Migrations/Sql/ChatProvedor_Idempotente.sql`
4. `backend/Central_BackEnd/Migrations/Sql/ChatConfiguracaoSistema_Idempotente.sql`

Atalho que aplica na ordem e confere o resultado (credencial por variável de
ambiente; exige `-ConfirmarBanco` com o nome exato):

```powershell
$env:CHAT_SQL_SERVIDOR='192.168.2.154'
$env:CHAT_SQL_BANCO='dbCENTRAL_OPERACAO'
$env:CHAT_SQL_USUARIO='...'
$env:CHAT_SQL_SENHA='...'
powershell -ExecutionPolicy Bypass -File .\scripts\aplicar-chat-sql.ps1 -ConfirmarBanco dbCENTRAL_OPERACAO
```

Somente conferir: acrescente `-SomenteConferir`.

- Os quatro scripts são idempotentes (seguros para reexecução).
- O segundo depende das tabelas criadas pelo primeiro (`tbchat_auditoria`).
- O segundo registra `20261022165354_ChatAgentes` em `__EFMigrationsHistory`.
- O quarto (`ChatConfiguracaoSistema`) cria `tbconfiguracao_sistema` e registra
  `20261030170000_ConfiguracaoSistema` em `__EFMigrationsHistory` quando a
  tabela já existe. `scripts\aplicar-chat-sql.ps1` confere a tabela ao final, e
  `scripts\preflight-schema.ps1` verifica `tbconfiguracao_sistema` e cita este
  SQL na mensagem de falha.

**Estado:** HML **corrigido e conferido em 2026-10-28** (faltava o `ciaa`; SQL
idempotente reaplicado com `-ConfirmarBanco dbBUSINESS_HML`); **PROD aplicado em
2026-10-28** com `scripts/aplicar-chat-sql.ps1`. Conferido nos dois: 6 agentes,
coluna `AGENTE`, `tbchat_workspace_funcao`, 2 migrations. Para reconferir sem
aplicar: `-SomenteConferir`.

---

## b) Conferências após o SQL

```sql
SELECT COUNT(*) FROM dbo.tbchat_agente;            -- esperado: 6
SELECT COL_LENGTH('dbo.tbchat_auditoria', 'AGENTE'); -- esperado: != NULL
SELECT OBJECT_ID('dbo.tbchat_workspace_funcao', 'U');-- esperado: != NULL
SELECT OBJECT_ID('dbo.tbconfiguracao_sistema', 'U'); -- esperado: != NULL (4º SQL)
```

Agentes esperados: `suporte`, `implantacao`, `desenvolvimento`, `financeiro`, `gestao`, `ciaa`.

---

## c) Segredos necessários (somente nomes)

Configurar via `dotnet user-secrets` (dev) ou variável de ambiente (servidor). Nunca versionar.

| Chave | Observação |
|---|---|
| `AnythingLLM:ApiKey` | Chave de integração do AnythingLLM. **O arquivo versionado (`backend/Central_BackEnd/appsettings.json`) tem o placeholder vazio**: no servidor use a variável de ambiente `AnythingLLM__ApiKey` (ou `user-secrets`). A chave que estava versionada era **inválida** (a API respondia 403) e foi substituída por uma nova, gerada no modo single-user e gravada **fora do repositório**. **PROD:** aplicada em 2026-10-28 no `web.config` do servidor (`web.config.bak-20261028-184627`); **HML:** aplicada no mesmo dia, no mesmo bloco (`web.config.bak-20261028-192936`) |
| `AnythingLLM:BaseUrl` | `http://192.168.2.253:3001/api/v1` (ou túnel local) |
| `Acessos:ChaveCifra` | **Opcional desde 2026-10-30**: a chave-mestra é gerada e gravada cifrada em `tbconfiguracao_sistema` na primeira subida. Se presente e base64 de 32 bytes (AES-256), tem precedência (override). O placeholder do `appsettings.json` é tratado como ausente |
| `Chat:Provedor` | `gemini` para teste imediato |
| `Gemini:Model` | Modelo usado pelo provedor Gemini |
| `Gemini:ApiKey` | **Chave nova** (a anterior foi exposta e deve ser rotacionada) |

> `Chat:Provedor=claude` exige `Claude:ApiKey` e `Claude:Model` e depende do fechamento da D28.
>
> **Sem `AnythingLLM__ApiKey` no app pool, a base do chat responde como
> "indisponível"** (não é falha do AnythingLLM): o status é medido por
> embeddings e erro de consulta vira `indisponivel`.
>
> **Troca de provedor:** o `switch` é resolvido no startup
> (`Configuration/ConhecimentoServiceCollectionExtensions.cs`). Trocar a chave ou o
> modelo do provedor atual vale no próximo request; **trocar o provedor exige
> recycle do app pool**. A chave da API do AnythingLLM usada na validação de
> 2026-10-28 também deve ser rotacionada.

---

## d) Validação

1. Gate completo:
   ```powershell
   powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
   ```
2. Backend — testes de Conhecimento (o chat antigo virou iframe/SSO):
   ```powershell
   dotnet test backend\Central_BackEnd.Tests --filter "FullyQualifiedName~ConhecimentoPathPolicyTests|FullyQualifiedName~ConhecimentoIntegracoesControllerTests"
   ```
3. Frontend — testes de Conhecimento:
   ```powershell
   cd frontend
   npx ng test --watch=false --browsers=ChromeHeadless --include="src/app/features/conhecimento/conhecimento-admin.component.spec.ts"
   ```

Não corrigir falhas fora do escopo sem consultar.

---

## e) Publicação

1. **IIS — reverse proxy** de `/api` e `/api/v1` para o backend .NET 8.
2. **runtime-config.json:** publicar `frontend/src/assets/config/runtime-config.json` com `apiBaseUrl` do ambiente (mesma origem do IIS em produção).
3. Confirmar que o navegador não acessa Gitea/AnythingLLM/Claude/n8n diretamente.

---

## f) Rede — porta 3001

O backend alcança o AnythingLLM em `http://192.168.2.253:3001/api/v1`. Se a regra
da porta 3001 for restrita, liberar **`192.168.2.254` (servidor IIS)** e
**`192.168.2.5` (estação de desenvolvimento)**. O navegador nunca fala com a 3001.

**Estado (2026-10-30):** a porta 3001 **já está liberada para a estação
(`192.168.2.5`)** — a API do AnythingLLM responde daí. O disparo do sync por
`POST /api/sync` acontece na **porta 8090** da VM, feito pela Central (`.254`/`.130`);
o ufw libera essa porta **apenas** para esses dois hosts. O header é
`X-Central-Sync-Key` e o segredo é o `KNOWLEDGE_SYNC_SECRET` da VM, com o mesmo valor em
`KnowledgeSync__Secret` nos dois backends.

## h) Segredos no banco — depois do 4º SQL e do deploy (2026-10-30)

Implementado em código e **SQL aplicado nos dois bancos em 2026-10-30**, ainda
**sem commit/deploy**. Ordem:

1. Conferir o 4º SQL (`ChatConfiguracaoSistema_Idempotente.sql`) nos dois bancos
   (já aplicado; reexecução idempotente).
2. Conferir no `preflight-schema.ps1` que `tbconfiguracao_sistema` existe
   (o publish aborta se faltar).
3. **Backup do diretório do chaveiro do DataProtection** —
   `CENTRAL_DATAPROTECTION_PATH` → `%ProgramData%\Central\DataProtection-Keys` →
   `App_Data/DataProtection-Keys` (fallback). Perder o chaveiro torna a
   chave-mestra do banco ilegível. Detalhe em
   [`11-10-preservacao-de-configuracao.md`](operacao/deploy/11-10-preservacao-de-configuracao.md).
4. Publicar backend e front e abrir `/conhecimento/configuracao` como admin.
5. Salvar o **token do Gitea** no bloco "Integração Gitea" (16..255 caracteres,
   sem espaço) → `204`; conferir a máscara (`••••abcd`) e a origem `banco`.
6. Conferir `GET /api/conhecimento/admin/integracoes`: `cifra_configurada=true`,
   `chave_mestra_origem=banco` (ou `configuracao` quando houver override) e
   `gitea_token_configurado=true`. O token **nunca** volta no JSON.
7. Negativos: token vazio/curto/com espaço → `400`; salvar sem cifra → `503`;
   não-admin → `403` em GET e PUT.

## g) Formato do `vector-search` (validado em 2026-10-28)

- `POST /api/v1/workspace/{slug}/vector-search` com `{ query, topN, scoreThreshold }`.
- Resposta: `{ results: [...], message }`; item = `id`, `text`, `metadata`, `score`.
- **Não existe `distance`** nesta versão.
- Workspace sem embeddings responde 200 com `results=[]` e
  `message="No embeddings found for this workspace."` — tratar como “sem fontes”.
- **Histórico:** o parser de `results` vivia em `ChatPerguntarService` e o status
  em `ChatAgenteStatusService` (removidos em `cb36ad3`); hoje o chat é um iframe
  do AnythingLLM com SSO (`GET /api/chat/sso`).
- **Validado na prática (2026-10-28)** no `glossario`:
  `POST /api/v1/workspace/glossario/vector-search` com
  `{query, topN:3, scoreThreshold:0}` → `results=3` (scores ~0,82).
- **Validado na prática (2026-10-30)** no `suporte`, com documento de
  `04-operacoes/`: `results=5` de **1 documento distinto** (scores 0,67–0,71) e
  scores idênticos após repetir o sync (sem duplicação). A instância segue
  **lenta**: ~3 min para indexar 1 documento em 4 workspaces.

## Fase 3 — sync único e trigger (concluída em 2026-10-30)

**Repositório:** `_config/workspaces.yml` versionado no `jca/jca-knowledge@main` (merge do PR #5,
commit `fd8b9b8`, blob `039a8ec9`). `financeiro` = `09-financeiro/` + `10-comercial/` (o workspace
`comercial` saiu do mapa); `07-revisao/` já está em `excluir` (a pasta nasce na Fase 4).

**Código (monorepo, não commitado):** `sync.py` lê o YAML pela API do Gitea
(`load_workspace_map`, parser próprio, fallback `WORKSPACE_ROOTS`), aceita `--workspaces a,b` e tem
timeouts ampliados (60/120/300/600 s); `test_sync.py` **13/13**. O `README.md` e as units
`systemd/*.service` apontam para `/opt/jca/sync/app`, `.env`, porta **8090** e
`X-Central-Sync-Key`.

**VM `192.168.2.253` (`/opt/jca/sync`, SSH `jca`):**

| Item | Estado |
|---|---|
| Código | `sync.py` e `trigger_server.py` em `/opt/jca/sync/app/`; `/opt/jca/sync/trigger_server.py` é launcher |
| `.env` (600) | usa `ANYTHINGLLM_API_KEY` (`central-backend`, 31 caracteres), **sem** a senha admin; `ANYTHINGLLM_PASSWORD`/`SYNC_TRIGGER_TOKEN` removidos; backup `.env.bak-fase3-20261030` |
| Manifesto | `MANIFEST_PATH=/opt/jca/sync/state/manifest.json`, semeado a partir do `state/glossario.json` (6 docs) |
| Trigger | `jca-sync-trigger.service` ativo, `0.0.0.0:8090`, `X-Central-Sync-Key`; `POST /api/sync` → 202 e `GET /api/status` → estado |
| Firewall | ufw com 8090/tcp para `.254` e `.130` (a regra antiga de `172.18.0.0/16` permanece) |
| Cron antigo | `sincronizar.sh` desativado (crontab vazio); arquivo mantido 1 semana para rollback |
| Estado da base | `glossario` = 6 docs; `suporte`/`implantacao`/`desenvolvimento`/`ciaa`/`financeiro` = 0; 2ª execução idempotente |

**Central (PROD e HML):** `web.config` dos dois backends atualizado via SMB com backup
`web.config.bak-fase3-20261030`, XML válido e UTF-8 sem BOM, com
`KnowledgeSync__TriggerUrl=http://192.168.2.253:8090/api/sync` e `KnowledgeSync__Secret` (mesmo
valor da VM; nunca impresso). **Aceite** verificado por HTTP autenticado nos dois:
`POST /api/conhecimento/sincronizar` → **202** `{"status":"agendado"}`;
`GET /api/conhecimento/sincronizacao` → `concluida`; contagem confere e a segunda execução não
duplicou.

## Rodada anterior — 2026-10-28 (operação, sem mudança de código)

- Chave do AnythingLLM trocada: a antiga (versionada) era **inválida** (403);
  a nova foi gerada no modo single-user e gravada **fora do repo**. O
  `appsettings.json` versionado ficou com placeholder **vazio**.
- `ciaa` **criado** no AnythingLLM (vazio: 0 documentos).
- `scripts/knowledge-sync/sync.py`: dry-run OK (6 docs de `06-glossario`);
  execução real interrompida por timeout do ambiente; duplicatas removidas — o
  `glossario` ficou com **exatamente 6 documentos**.
- `vector-search` **provado** no `glossario` (`results=3`).
- Gitea: teste real de escrita OK (branch temporária + commit, autor `jca`,
  `main` intacta). Token de leitura (`central-sync-read`) gravado fora do repo.
- OAuth do Gitea: **ausente** (`Gitea:OAuth:ClientId/ClientSecret` vazios) ⇒ o
  login do operador na Central continua indisponível.

## PROD — configurado e verificado (2026-10-28)

**O que foi configurado** (sem deploy; via SMB `\\192.168.2.254\c$`):

| Item | Onde | Estado |
|---|---|---|
| Env vars do backend | `inetpub\wwwroot\CentralOperacao\Back\web.config`, no mesmo bloco `environmentVariables` já usado pelo WhatsApp | `AnythingLLM__BaseUrl`, `AnythingLLM__ApiKey`, `AnythingLLM__TimeoutSeconds` (30), `AnythingLLM__StatusTimeoutSeconds` (8), `Gitea__BaseUrl/Owner/Repository/MainBranch`, `Gitea__OAuth__AuthorizeUrl/TokenUrl/ClientId/ClientSecret/CallbackUrl/FrontendUrl`, `KnowledgeSync__TriggerUrl` |
| Backup da config anterior | mesmo servidor | `web.config.bak-20261028-184627` |
| App OAuth2 do Gitea | Gitea (`192.168.2.253:3000`) | criado (`jca`) com os redirect URIs do callback do backend |
| Segredos | **fora do repositório** | `client_id`/`client_secret` do Gitea e a chave do AnythingLLM existem **só** no servidor e em `.opencode/secrets/` (gitignored) — **nenhum valor neste arquivo** |

Alterar o `web.config` **reinicia a aplicação no IIS** (não é deploy).

**Como verificar** (de dentro do PROD; token JWT de verificação emitido com a
`Jwt:Key` do próprio servidor — **nenhuma senha de operador foi lida ou
alterada**):

| Chamada | Esperado (2026-10-28) |
|---|---|
| `GET /api/chat/agentes` | **200**, 6 agentes (suporte, implantacao, desenvolvimento, financeiro, gestao, ciaa), todos `base_status=em_construcao` — o backend alcança o AnythingLLM (sem chave seria `indisponivel`) |
| `POST /api/conhecimento/gitea/oauth/iniciar` | **200** com a authorize URL do Gitea |
| `GET /api/conhecimento/permissao` | **200** `{pode_acessar:true, eh_revisor:true, eh_admin:true, workspaces:[suporte,financeiro,implantacao,comercial,glossario], gitea_conectado:false}` |

`gitea_conectado:false` é o esperado: falta **conectar a conta do operador** no
navegador.

## HML — configurado e publicado (2026-10-28)

O HML **deixou de estar pendente**. O `web.config` do servidor
(`inetpub\wwwroot\Suporte_Back\web.config`) tinha só `WhatsApp__*`; como esse
marcador existe, o publish **preservou** o arquivo (cópia datada
`web.config.pre-deploy-20261028-192811`) e o bloco de env vars do chat foi
gravado **via SMB** com a conta de serviço do deploy (`JCASRV-SUP`).

**Bloco gravado (idêntico ao do PROD):**

| Grupo | Variáveis |
|---|---|
| AnythingLLM | `AnythingLLM__BaseUrl`, `AnythingLLM__ApiKey`, `AnythingLLM__TimeoutSeconds` (30), `AnythingLLM__StatusTimeoutSeconds` (8) |
| Gitea | `Gitea__BaseUrl`, `Gitea__Owner`, `Gitea__Repository`, `Gitea__MainBranch` |
| Gitea OAuth | `Gitea__OAuth__AuthorizeUrl`, `Gitea__OAuth__TokenUrl`, `Gitea__OAuth__ClientId`, `Gitea__OAuth__ClientSecret`, `Gitea__OAuth__CallbackUrl`, `Gitea__OAuth__FrontendUrl` |
| Sync | `KnowledgeSync__TriggerUrl` |

| Item | Onde | Estado |
|---|---|---|
| Env vars do backend | `inetpub\wwwroot\Suporte_Back\web.config` | bloco acima gravado — **20 env vars** no total |
| Backup da config anterior | mesmo servidor | `web.config.bak-20261028-192936` |
| Cópia do publish | mesmo servidor | `web.config.pre-deploy-20261028-192811` |
| `appsettings.json` do servidor | mesmo servidor | **sem segredo** (placeholder vazio), como no PROD |
| Schema do chat | `192.168.2.154/dbBUSINESS_HML` | recheck acusou “FALTA 6 agentes no catalogo (encontrado: 5)”; SQL idempotente reaplicado com `-ConfirmarBanco dbBUSINESS_HML` → **6 agentes** |
| Deploy | `192.168.2.130` | commit `f58cb6e` publicado; backup `\\192.168.2.130\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261028-192800` |

**Smoke do HML (2026-10-28):** front 200, swagger 200, runtime-config OK,
login 401 (esperado) e webhook do WhatsApp 401 (esperado). **500** nesses
pontos = schema/código quebrado.

**Verificação funcional** (de dentro do servidor; token emitido com a `Jwt:Key`
do próprio servidor — **nenhuma senha de operador foi lida ou alterada**):
`GET /api/chat/agentes` → **200** com 6 agentes em `base_status=em_construcao`
(o backend alcança o AnythingLLM); `POST /api/conhecimento/gitea/oauth/iniciar`
→ **200** com a authorize URL do Gitea; `GET /api/conhecimento/permissao` →
**200** com `eh_revisor`/`eh_admin` = `true` e `gitea_conectado=false`.

### Verificação por ambiente (comando e o que esperar)

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1 -Servidor 192.168.2.130   # HML
powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1 -Servidor 192.168.2.254   # PROD
```

| Ambiente | Servidor | Esperado no smoke |
|---|---|---|
| HML | `192.168.2.130` | front 200, swagger 200, runtime-config OK, login 401, webhook do WhatsApp 401 |
| PROD | `192.168.2.254` | front 200, swagger 200, runtime-config OK, login 401, webhook do WhatsApp 401 |

Nos dois: `/chat?vista=agentes` mostra os 6 cards e
`GET /api/chat/agentes` responde **200** com `base_status=em_construcao` (base
alcançável, ainda sem embeddings). O bundle publicado traz a tela de Agentes
(marcadores “Base: disponível”, “Em breve” e `vista=agentes` presentes).

**Armadilha de deploy (2026-10-28):** `scripts/deploy/deploy-env.ps1` e
`scripts/deploy/deploy.ps1` declaram `[Nullable[bool]]` em
`-BuildFrontend`/`-BuildBackend`. Chamados por `-File` com `1`/`0` falham com
erro de transformação de argumento — chame no **próprio PowerShell** com
`$true`/`$false`.

## Fora do código (depende de pessoas)

- **Corpus (2026-10-30)**: `07-revisao/` concentra os documentos revisados
  (147 migrados na Fase 4; 146 após a promoção da Fase 5) e `08-pendente/revisar/`
  tem só `.gitkeep`. `01-empresa` a `05-infraestrutura` continuam com `.gitkeep`
  porque o sync indexa o documento aprovado, não a pasta inteira.
- **Fase 4 (concluída em 2026-10-30)**: PR #6, squash `088611c` — 147 renames para
  `07-revisao/`, `.gitkeep` em `09-financeiro/` (4 subpastas) e `10-comercial/`,
  `MATRIZ-SETOR-WORKSPACE.md` atualizada.
- **Fase 5 Caminho B (parcial)**: 1 documento promovido para `04-operacoes/`
  (commit `87ab8bd`, autor `jca`), sync na VM concluída — manifesto com 7
  documentos (glossario 6; suporte, implantação, desenvolvimento e ciaa com 1).
  Falta o teste por `POST /api/chat/perguntar` (exige JWT de operador).
- **Aplicar as Fases 1 e 2**: SQL/migration de `tbchat_provedor` e publicação nos dois ambientes;
  as fases seguem implementadas só em código/testes. Inclui o 4º SQL
  (`ChatConfiguracaoSistema_Idempotente.sql`, tabela `tbconfiguracao_sistema`).
- **Conectar a conta do operador no Gitea** (navegador) no HML e no PROD: o
  OAuth já está ativo nos dois ambientes; `gitea_conectado=false` até lá.
- **Token de serviço do Gitea com escrita**: salvar pela tela admin
  (`/conhecimento/configuracao` → "Integração Gitea") depois do deploy, em
  HML e PROD; sem token (nem banco nem `Gitea__ServiceToken`) a aprovação
  responde `503`. A env var continua válida como fallback.
- Rotacionar a chave do Gemini exposta e a chave `central-backend` do AnythingLLM.
- Conferir na **UI admin** do AnythingLLM que agentes/web/scripts/MCP estão
  desativados (nesta versão não há endpoint de feature flags).
- Fechar decisões D11, D20, D21 e D28.
