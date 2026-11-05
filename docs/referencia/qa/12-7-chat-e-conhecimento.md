---
id: sec.12-7-chat-e-conhecimento
titulo: "7. Chat e Conhecimento"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
  - backend/Central_BackEnd.Tests/ConhecimentoPathPolicyTests.cs
  - backend/Central_BackEnd.Tests/ConfiguracaoSistemaTests.cs
  - backend/Central_BackEnd.Tests/GiteaTokenServiceTests.cs
  - backend/Central_BackEnd.Tests/ConhecimentoIntegracoesControllerTests.cs
  - frontend/src/app/features/conhecimento/conhecimento-admin.component.spec.ts
tokens_estimados: 4200
ficha: docs/modulos/chat-conhecimento.md
---
> Ficha: [Chat e Conhecimento](../../modulos/chat-conhecimento.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 7. Chat e Conhecimento

> **Histórico (2026-11-03):** a arquitetura anterior do chat (`/api/chat/agentes`,
> `/api/chat/perguntar`, `JotaChatService`, widget global) foi removida no `cb36ad3`.
> Hoje o chat é iframe AnythingLLM autenticado por `GET /api/chat/sso`.
> Os cenários CJ-01, CJ-01b, CJ-07, CJ-03 e CJ-04 foram reescritos ou marcados
> como descontinuados. Conhecimento válido.

### CJ-01 — Abrir o chat (iframe + SSO)

- **Objetivo:** abrir `/chat` e ver o AnythingLLM carregado em iframe.
- **Pré-condições:** operador ativo; backend no ar; AnythingLLM configurado no ambiente.
- **Passos:** 1. Abrir `/chat`. 2. Aguardar a carga. 3. Provocar falha (sessão expirada ou backend fora) e observar a tela.
- **Resultado esperado:** `GET /api/chat/sso` devolve `{ url }`; o iframe carrega com `allow="microphone"` e título "Chat"; sem `url`/em erro, a tela mostra o estado de carregamento ou a mensagem de erro ("Não foi possível obter a URL do chat." / `mensagem` do backend / "Falha ao carregar o chat."). Nunca 500 silencioso.
- **Estado atual/bloqueio:** publicado em HML (.130) e PROD (.254); SQL do chat aplicado em 2026-10-30.
- **Rotas/endpoints:** `/chat`; `GET /api/chat/sso` (`ChatSsoController`); componente `ChatIframeComponent`.
- **Status/automação:** manual (smoke dos dois ambientes); spec do componente inexistente — comportamento visual é manual.

### CJ-01b — Vista "Agentes" dentro de /chat (descontinuado em 2026-11-03)

- **Estado atual/bloqueio:** cenário descontinuado. A vista `/chat?vista=agentes` foi removida
  junto com o catálogo de agentes (`cb36ad3`).

### CJ-07 — Falha de carregamento do chat é visível (descontinuado em 2026-11-03)

- **Estado atual/bloqueio:** cenário absorvido pelo CJ-01 (iframe + SSO); o front não chama
  mais catálogo nem `JotaChatService` — a falha aparece como erro na própria página `/chat`.

### CJ-03 — Ambiente indisponível degrada para erro visível (reescrito em 2026-11-05)

- **Objetivo:** backend/AnythingLLM indisponível mostra erro, nunca 500 cru.
- **Pré-condições:** `appsettings.json` versionado sem chaves; os dois ambientes com variáveis no `web.config`.
- **Passos:** abrir `/chat` com o AnythingLLM configurado e com a chave inválida.
- **Resultado esperado:** sem configuração, `GET /api/chat/sso` responde erro com `mensagem`; a tela exibe o erro. Nunca expõe segredo nem responde 500 sem corpo.
- **Estado atual/bloqueio:** chave válida nos dois `web.config`; validado no deploy de 2026-10-28.
- **Rotas/endpoints:** `GET /api/chat/sso`.

### CJ-04 — Workspace vazio não é base pronta (descontinuado em 2026-11-03)

- **Estado atual/bloqueio:** descontinuado — a criação/verificação de workspaces acontece no
  AnythingLLM/admin do conhecimento, não pela Central.

### CJ-08 — Trocar provedor de IA (removido em 2026-11-03)

- **Estado atual/bloqueio:** cenário descontinuado. O chat passou a ser um iframe
  do AnythingLLM (`GET /api/chat/sso`) e o painel "Provedor de IA" foi removido
  do front junto com os endpoints `GET/PUT /api/chat/provedor` e
  `POST /api/chat/provedor/testar`. Mantido apenas como histórico.

### CJ-11 — Bloco "Integração Gitea" (token cifrado no banco)

- **Objetivo:** admin salva o token de serviço do Gitea pela tela, o token fica cifrado no banco e a UI mostra apenas máscara/origem.
- **Pré-condições:** admin; 4º SQL aplicado (`ChatConfiguracaoSistema_Idempotente.sql` → `tbconfiguracao_sistema`); backend publicado; cifra disponível (chave-mestra no banco ou `Acessos__ChaveCifra` como override); tela `/conhecimento/configuracao`.
- **Passos:** 1. Abrir o bloco e conferir máscara, origem e botão. 2. Confirmar `password` write-only. 3. Salvar token válido (16..255, sem espaço) e recarregar (máscara nova, origem `banco`). 4. Tentar vazio/curto/com espaço. 5. Repetir sem cifra e como não-admin.
- **Resultado esperado:** `GET /api/conhecimento/admin/integracoes` → `200` com `gitea_token_configurado`, `gitea_token_mascara`, `gitea_token_origem`, `cifra_configurada` e `chave_mestra_origem`; **nunca** o token. `PUT /api/conhecimento/admin/integracoes/gitea` → `204` no sucesso; `400` para token vazio/fora de 16..255/com espaço; `503` sem cifra; `403` para não-admin nos dois.
- **Estado atual/bloqueio:** implementado em código/testes (2026-10-30); SQL aplicado nos dois bancos; publicado em HML e PROD (2026-10-28).
- **Rotas/endpoints/tabelas:** `/conhecimento/configuracao`; `GET /api/conhecimento/admin/integracoes`, `PUT /api/conhecimento/admin/integracoes/gitea`; `tbconfiguracao_sistema` (`acessos.chave_cifra`, `gitea.service_token`).
- **Status/automação:** `ConhecimentoIntegracoesControllerTests` (5), `GiteaTokenServiceTests` (5), `ConfiguracaoSistemaTests` (5) e `conhecimento-admin.component.spec.ts`; suíte de Chat/Conhecimento 64/64 verde.

### CJ-09 — Aprovar escolhendo a pasta de destino

- **Objetivo:** revisor aprova documento do staging escolhendo destino (`01..06`, `09-financeiro/`, `10-comercial/`).
- **Pré-condições:** token de serviço do Gitea com escrita — salvo pela tela admin (`tbconfiguracao_sistema`) ou `Gitea__ServiceToken` (**bloqueio do aceite**); documento em `07-revisao/` (legado `08-pendente/revisar/` legível); revisor ou admin.
- **Passos:** 1. Clicar **Aprovar**: modal com seletor, default = `pasta_sugerida`. 2. Escolher e confirmar. 3. Tentar destino de staging (`07-revisao/`/`08-pendente/`) ou inválido. 4. Aprovar sem escolha e sem `pasta_sugerida`.
- **Resultado esperado:** arquivo movido para a pasta escolhida (escolha vence a sugestão), com auditoria, `INDEX` atualizado e sync agendado; staging como destino → `400`; sem escolha nem sugestão → `400`; sem token → `503`.
- **Estado atual/bloqueio:** implementado em código/testes (Fase 2); aceite depende do token de serviço.
- **Rotas/endpoints/tabelas:** `/conhecimento/editor`; `POST /api/conhecimento/aprovar` (`pasta_destino`); `tbconhecimento_auditoria`.
- **Status/automação:** `ConhecimentoPathPolicyTests` cobre `07`/`09`/`10`, staging proibido e escolha/sugestão/ausência.

### CJ-10 — Mapa volátil e sync disparado pela Central

- **Objetivo:** conferir que o mapa pasta→workspace vem de `_config/workspaces.yml` (editável por PR, sem deploy) e que a Central dispara o sync e vê o estado.
- **Pré-condições:** `_config/workspaces.yml` no `jca/jca-knowledge@main`; VM `.253` com `sync.py` em `/opt/jca/sync/app` e `jca-sync-trigger.service` ativo na 8090; `KnowledgeSync__TriggerUrl`/`KnowledgeSync__Secret` nos dois backends; operador admin.
- **Passos:** 1. Conferir no YAML `financeiro` = `09-financeiro/` + `10-comercial/` e `07-revisao/` em `excluir`. 2. Na VM, rodar `sync.py --dry-run` e ler a origem do mapa. 3. Chamar `POST /api/conhecimento/sincronizar` e `GET .../sincronizacao`. 4. Repetir e comparar contagens. 5. Rodar `--workspaces glossario` e conferir o checkpoint dos demais.
- **Resultado esperado:** `202 agendado` no disparo e `concluida` no status; a segunda execução não duplica; pasta fora do mapa/`excluir` não é indexada.
- **Estado atual/bloqueio:** concluído em 2026-10-30 (PR #5; `test_sync.py` 13/13; aceite em PROD/HML).
- **Rotas/endpoints/tabelas:** `POST /api/conhecimento/sincronizar`, `GET /api/conhecimento/sincronizacao`; `POST /api/sync`/`GET /api/status` (VM, 8090).
- **Status/automação:** `scripts/knowledge-sync/test_sync.py` cobre YAML, exclusão de `07-revisao/`, financeiro/comercial e fallback; disparo e status são manuais.

### CJ-05 — Sync duplicando documentos

- **Objetivo:** detectar e eliminar duplicatas depois do sync.
- **Pré-condições:** API de desenvolvedor do AnythingLLM e repositório `jca/jca-knowledge`.
- **Passos:** 1. Listar por workspace (`GET /api/v1/workspace/{slug}`) e conferir `docpath` (glossario = 6). 2. Repetido? Remover com `POST /api/v1/workspace/{slug}/update-embeddings` + `deletes` (1 registro por `docpath` por chamada). 3. Reconferir.
- **Resultado esperado:** um registro por documento. Causas: `MANIFEST_PATH` divergente da VM e retry em 5xx.
- **Estado atual/bloqueio:** duplicatas removidas em 2026-10-28. Sync oficial na VM `.253` (`/opt/jca/sync/app`, manifesto fixo, trigger 8090); em 2026-10-30 o `glossario` tinha 6 documentos e a reexecução não duplicou.
- **Rotas/endpoints:** `GET /api/v1/workspace/{slug}`, `POST .../update-embeddings`, `POST /api/sync` (VM, 8090).

### CJ-06 — Ambiente configurado (PROD × HML)

- **Objetivo:** conferir a configuração de cada ambiente pela API.
- **Pré-condições:** PROD (`192.168.2.254`) e HML (`192.168.2.130`) com `web.config` (`AnythingLLM__*`, `Gitea__*`, `KnowledgeSync__TriggerUrl`/`KnowledgeSync__Secret`); SQL do chat aplicado (2026-10-30).
- **Passos:** em cada ambiente, chamar `GET /api/chat/sso` e `GET .../permissao`; rodar `smoke.ps1 -Servidor <IP>`.
- **Resultado esperado:** `GET /api/chat/sso` devolve `{ url }` (200 autenticado; 401 sem sessão); OAuth → 200 com authorize URL; `permissao` → 200 (`pode_acessar`, `eh_revisor`, `eh_admin`, `gitea_conectado=false`). Smoke: front/swagger/runtime-config 200, login **401** e webhook **401** (500 = quebrado).
- **Estado atual/bloqueio:** configurados e verificados (2026-10-28; publicação em HML e PROD); falta conectar a conta no Gitea.
- **Rotas/endpoints:** `GET /api/chat/sso`, `POST /api/conhecimento/gitea/oauth/iniciar`, `GET /api/conhecimento/permissao`.

### CJ-02 — Consultar e revisar Conhecimento

- **Objetivo:** verificar o gate de revisor e o fluxo de documentos em Gitea.
- **Pré-condições:** revisor associado a uma função (ou admin); migration aplicada; OAuth Gitea e token de serviço no servidor; conta pessoal autorizada.
- **Passos:** 1. Abrir `/conhecimento`, confirmar `permissao` e listar/filtrar a árvore. 2. Abrir documento, criar revisão, consultar diff/histórico e forçar conflito de SHA. 3. Conectar OAuth, aprovar um documento e rejeitar outro. 4. Como admin, abrir a configuração, solicitar sync e ver o status.
- **Resultado esperado:** escopo de workspace respeitado; conflito `409` traz SHA/conteúdo atuais sem sobrescrever. Aprovação/rejeição usa token de serviço para o PR e OAuth pessoal para aprovar/mesclar; rejeição mantém `status: rejeitado` no caminho original. Sync manual por `POST /api/conhecimento/sincronizar` e `GET /api/conhecimento/sincronizacao` (contrato em [`16-conhecimento-e-revisao-documental.md`](../endpoints/raiz/16-conhecimento-e-revisao-documental.md)).
- **Estado atual/bloqueio:** migration/SQL aplicados (2026-10-28; SQL do chat em 2026-10-30); OAuth e segredos no `web.config` dos dois (falta conectar o operador); sync/trigger concluídos (VM, 8090); publicado em HML e PROD.
- **Rotas/endpoints/tabelas:** `/conhecimento{,/nova,/configuracao,/editor,/diff,/historico}`; `POST /conhecimento/sincronizar`, `GET /conhecimento/sincronizacao`, `/api/sync`, `/api/status`; `tbchat_workspace_funcao`, `tbconhecimento_revisor_funcao`, `tbconhecimento_credencial_gitea`, `tbconhecimento_auditoria`.
- **Status/automação:** `revisor.guard.spec.ts` cobre permitido/negado; o restante é manual.
