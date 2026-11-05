---
id: sec.12-11-chat-e-conhecimento
titulo: "11. Chat e Conhecimento"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 3000
ficha: docs/modulos/chat-conhecimento.md
---
> Ficha: [Chat e Conhecimento](../../modulos/chat-conhecimento.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 11. Chat e Conhecimento

**Contrato ativo (desde `cb36ad3`, 2026-11-03):** o chat é **iframe para o AnythingLLM** autenticado por `GET /api/chat/sso` (`ChatSsoController`); a página `/chat` (`ChatIframeComponent`) busca `{ url }` e o navegador carrega o AnythingLLM. `/api/conhecimento/*` e o SSO seguem sem versão. `/api/rag-proxy`, `/api/chat/agentes`, `/api/chat/perguntar` e `/api/chat/provedor` foram **removidos**. O conteúdo é processado pelo AnythingLLM e pelos provedores configurados nele; a Central não grava pergunta/resposta. O agente/provedor de IA configurado na Central foi descontinuado — as tabelas `tbchat_agente`, `tbchat_auditoria`, `tbchat_provedor` e `tbchat_workspace_funcao` permanecem no banco sem consumidor.

**Estado (publicado 2026-10-28; SQL aplicado em 2026-10-30).** PROD (`192.168.2.254`) e HML (`192.168.2.130`) configurados e publicados: `AnythingLLM__ApiKey`, `Gitea__*` e `KnowledgeSync__TriggerUrl` no `web.config` de cada um; o `appsettings.json` versionado segue sem segredo. De dentro de cada servidor: `GET /api/chat/sso` → 200 com `{ url }`, `gitea/oauth/iniciar` → 200 e `permissao` → 200 (`eh_revisor`/`eh_admin`). OAuth ativo nos dois; falta conectar a conta do operador (`gitea_conectado=false`). SQL das 4 migrations de chat aplicado em HML e PROD em 2026-10-30.

**Sync e trigger (Fase 3, concluída em 2026-10-30).** O `sync.py` do repositório roda na VM `.253` em `/opt/jca/sync/app/` e lê o mapa pasta→workspace de `_config/workspaces.yml` (fallback embutido, com a origem no log); manifesto fixo em `/opt/jca/sync/state/manifest.json`. O trigger é o `jca-sync-trigger.service` (systemd ativo), escuta em `0.0.0.0:8090`, exige `X-Central-Sync-Key` e responde 202 em `POST /api/sync`; `GET /api/status` devolve o estado. O ufw libera 8090/tcp apenas para `.254` e `.130`. O `.env` usa `ANYTHINGLLM_API_KEY` (sem a senha admin) e `KNOWLEDGE_SYNC_SECRET`, com o mesmo valor em `KnowledgeSync__Secret` nos dois backends; o cron do `sincronizar.sh` foi desativado (arquivo mantido 1 semana para rollback). Aceite verificado: `POST /api/conhecimento/sincronizar` → 202 e `GET /api/conhecimento/sincronizacao` → `concluida` em PROD e HML, sem duplicar na segunda execução.

**Incidente do front (2026-10-28).** A condição que excluía `/api/*` do fallback SPA (`7ac8c61`) derrubou os dois sites (`500.50.13`) e foi revertida (`a5e2df6`); não é requisito do chat. Restaurar `web.config` com `>` do PowerShell grava UTF-16 e gera `500.19` — use bytes crus (`cmd /c`). Detalhes em [`12-11-backup-e-rollback.md`](../deploy/12-11-backup-e-rollback.md) e [`11-10-preservacao-de-configuracao.md`](../deploy/11-10-preservacao-de-configuracao.md).

**Guard e pendências.** O guard Angular é só navegação; o servidor valida permissão e escopo. Antes de rejeitar, `_config/status-documento.yml` precisa aceitar `rejeitado`. A aprovação depende de `Gitea__ServiceToken` (sem ele a aprovação responde `503`). O que resta de infraestrutura é o encaminhamento IIS/reverse proxy: não presuma ARR só porque o backend escuta em 1009.

### Diagnóstico e mitigação

| Sintoma | Causa provável | Mitigação |
|---|---|---|
| `/api/conhecimento` ou `/api/chat/sso` retorna HTML da SPA | site do front devolve `index.html` em `/api/*` (fallback normal) ou proxy ausente | o front deve chamar a API na origem do `runtime-config` (`:1009`); confira o bundle e a `runtime-config.json`; se precisar, configure proxy externo para o backend |
| Chat mostra **erro ao carregar** ("Não foi possível obter a URL do chat." / "Falha ao carregar o chat.") | `GET /api/chat/sso` falhou ou devolveu vazio | (1) confira `/api/chat/sso` no backend → 200 com `{ url }`; (2) bundle publicado com `apiRootUrl` — se vier HTML no próprio origin, é bundle antigo → Ctrl+F5/republicar; (3) confira a `runtime-config.json`; (4) HTML em `/api/*` no front é fallback normal, não defeito |
| **Front inteiro em 500** (`500/50/13`) ou **500.19** | URL Rewrite inválido no `web.config` do front ou arquivo em UTF-16 | restaure em bytes crus (`cmd /c "git show <commit>:frontend/public/web.config > arquivo"`) ou cópia binária; nunca `>` do PowerShell; valide com `scripts/smoke.ps1` |
| `GET /api/chat/sso` retorna 401 | Bearer ausente ou sessão expirada | valide login/refresh e o caminho relativo/encaminhamento |
| iframe não carrega / `502` | AnythingLLM sem `BaseUrl`/`ApiKey`, fora do ar ou com resposta inesperada | confira a configuração `AnythingLLM:*` e o log do `ChatSsoController`; `GET /api/chat/sso` deve responder `200` com `url` |
| Chat retorna 502/504 | AnythingLLM indisponível, segredo/URL/modelo ausente ou timeout | valide conectividade e configuração protegida; não registre pergunta ou chaves |
| Aprovar documento responde **503** | `Gitea:ServiceToken` ausente: sem token de escrita não há PR | configure `Gitea__ServiceToken` no `web.config` dos dois e recycle; o token nunca vai ao front |
| Aprovar responde **400** | destino inválido (staging `07-revisao/`/`08-pendente/`, `..` ou absoluto) ou nenhuma pasta (sem `pasta_destino` e sem `pasta_sugerida`) | escolha `01..06`, `09-financeiro/` ou `10-comercial/`; a escolha do revisor vence o frontmatter |
| Documento de revisão ausente da árvore | staging definido por `Conhecimento:PastaRevisao` (default `07-revisao/`); o legado `08-pendente/revisar/` continua listado | a árvore mostra staging configurado + legado + Financeiro/Comercial; `sync.py` ignora o staging |
| tabela inexistente `tbchat_*`/`tbconhecimento_*` | migration ou script idempotente ausente | confirme `__EFMigrationsHistory` e aplique o script aprovado antes do publish; o deploy não roda DDL |
| Conhecimento retorna 403 ou pede conexão Gitea | sem vínculo de revisor, OAuth pessoal não concluído ou conta sem permissão no repo | confira `GET /api/conhecimento/permissao`, vínculo e OAuth pessoal; o token de serviço não substitui a identidade pessoal |
| OAuth redireciona para estado inválido | cache de state expirado (5 min) ou instâncias/processos distintos | reinicie o OAuth e mantenha o callback no mesmo processo/instância |
| Chat não autentica **só no HML** | `web.config` do HML sem `AnythingLLM__*`/`Gitea__*`/`KnowledgeSync__*` | grave o mesmo bloco do PROD no `192.168.2.130` com backup datado e recycle; sem `AnythingLLM__ApiKey` o iframe não carrega e sem `Gitea__OAuth__*` o OAuth não abre |
| Documentos duplicados (mesmo `docpath`) | `MANIFEST_PATH` divergente da VM ou retry de upload em 5xx | conte por workspace (`GET /api/v1/workspace/{slug}`) e remova com `POST .../update-embeddings` (`deletes` remove 1 por `docpath` por chamada) |
| Sync reenvia **todos** os documentos | o checkpoint gravava o sha256 antigo; execução interrompida reenviava tudo | corrigido no `sync.py` (`manifest_entry` grava o digest atual); se já duplicou, limpe antes de reindexar |
| Retry de upload (5xx) deixa várias linhas | `update-embeddings` remove 1 registro por `docpath` por chamada | corrigido com `purge_location` (remove cópias até `keep`); manualmente, repita a chamada |
| Sync diz "outra execução já está ativa" | lock órfão de processo morto (fallback Windows) | lock mais velho que `SYNC_LOCK_MAX_AGE_MINUTES` (30 min) é assumido com aviso; apague o `.lock` sem execução viva |
| Trigger não responde | serviço `jca-sync-trigger` parado, porta 8090 não liberada para o host ou header ausente/errado | na VM: `systemctl status jca-sync-trigger`, `ss -ltnp` (0.0.0.0:8090) e `ufw status`; da Central, `POST /api/conhecimento/sincronizar` e `GET /api/conhecimento/sincronizacao`. O header é `X-Central-Sync-Key` |
| Sync não indexa uma pasta | pasta fora do `_config/workspaces.yml`, em `excluir`, ou execução limitada por `--workspaces` | confira o mapa no Gitea (editável por PR) e o log do `sync.py`, que informa se o mapa veio do YAML ou do fallback |
| OAuth do Gitea ausente | `Gitea:OAuth:ClientId/ClientSecret` vazios | `gitea/oauth/iniciar` → 503; configure o segredo no servidor |
| OAuth responde 401/403 no callback | `Gitea__OAuth__ClientId/ClientSecret/CallbackUrl` divergentes ou redirect URI não cadastrado | os valores vão no `web.config`; o callback do Gitea deve ser exatamente o `Gitea__OAuth__CallbackUrl` |
| Sync permanece pendente/erro | `KnowledgeSync__TriggerUrl`/`Secret` ausentes ou divergentes, firewall, serviço parado ou VM indisponível | confira `GET /api/conhecimento/sincronizacao`; admin dispara `POST /api/conhecimento/sincronizar`. Na VM, o trigger é `jca-sync-trigger.service` na 8090 e `GET /api/status`/`POST /api/sync` exigem `X-Central-Sync-Key`; verifique `sync.log`/`trigger.log` e mantenha a porta restrita a `.254`/`.130` |

Para confirmar o caminho no navegador, observe URL/status sem copiar Bearer, segredo, pergunta, resposta ou conteúdo de documentos para logs.
