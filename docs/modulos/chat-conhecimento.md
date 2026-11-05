---
id: mod.chat-conhecimento
titulo: "Chat e Conhecimento"
tipo: modulo
area: conhecimento
sistema: central
tags: [chat, rag, conhecimento, documento, revisao, gitea, anythingllm, sso, tbdoc]
dono: conhecimento
status: ativo
revisado_em: "2026-11-03"
fonte: [backend/Central_BackEnd/Controllers/ChatSsoController.cs, backend/Central_BackEnd/Controllers/ConhecimentoController.cs, backend/Central_BackEnd/Configuration/ConhecimentoServiceCollectionExtensions.cs, backend/Central_BackEnd/Services/Configuracao/ConfiguracaoSistemaService.cs, backend/Central_BackEnd/Services/Configuracao/ChaveMestraProvider.cs, backend/Central_BackEnd/Services/Conhecimento, frontend/src/app/features/chat/chat-iframe.component.ts, frontend/src/app/features/conhecimento, frontend/src/app/core/services/config.service.ts]
testes: [backend/Central_BackEnd.Tests/ConhecimentoPathPolicyTests.cs, backend/Central_BackEnd.Tests/ConfiguracaoSistemaTests.cs, backend/Central_BackEnd.Tests/GiteaTokenServiceTests.cs, backend/Central_BackEnd.Tests/ConhecimentoIntegracoesControllerTests.cs, frontend/src/app/features/conhecimento/conhecimento-admin.component.spec.ts, scripts/knowledge-sync/test_sync.py]
tokens_estimados: 1900
---

# Chat e Conhecimento

## 1. Quando usar este card

Chat embutido (AnythingLLM) e revisão de documentos do Conhecimento.

## 2. Propósito e escopo

**Chat** é um iframe do AnythingLLM autenticado por SSO. **Conhecimento** é a
revisão dos documentos versionados. Não cobre o Gitea fora da Central nem o
`docs/`.

## 3. Rotas do frontend

`/chat` (`chat-iframe.component.ts`, carregado em `features.routes.ts`),
`/conhecimento`, `/conhecimento/nova`, `/conhecimento/configuracao`,
`/conhecimento/editor`, `/diff`, `/historico`. `/conhecimento` usa
`revisorGuardLazy` e o editor usa `unsavedChangesGuard`.

## 4. Endpoints do backend

Todos `[Authorize]`.

- `GET /api/chat/sso` — `backend/Central_BackEnd/Controllers/ChatSsoController.cs`.
  Lê o `sub` do JWT, procura o usuário no AnythingLLM (`GET /admin/users`),
  emite token (`GET /users/{id}/issue-auth-token`) e devolve
  `{ loginPath, url, token }`, onde `url` é a URL pronta para o iframe.
  Erros: `401` sem identidade, `404` operador sem conta no AnythingLLM,
  `502` falha/configuração ausente, `504` timeout.
- `GET /api/conhecimento/{permissao,arvore,documento,diff,historico,sincronizacao}`;
  `PUT/POST /api/conhecimento/{documento,documento/nova,aprovar,rejeitar}`;
  `POST /conhecimento/sincronizar`; `GET/POST /gitea/oauth/*`;
  `PUT /conhecimento/admin/{workspaces,revisores}` —
  `backend/Central_BackEnd/Controllers/ConhecimentoController.cs`.
- `GET /api/conhecimento/admin/integracoes` e
  `PUT /api/conhecimento/admin/integracoes/gitea` — mesmo controller, somente
  admin. O GET devolve `gitea_token_configurado`, `gitea_token_mascara`,
  `gitea_token_origem`, `cifra_configurada` e `chave_mestra_origem`; nunca o
  token. Contratos em [endpoints](../referencia/endpoints.md).

> `/api/chat` é exceção sem versão; por isso o front usa `apiRootUrl` (host puro),
> não `apiBaseUrl`.

## 5. Tabelas e entidades

Permanecem no schema: `ChatAgente` → `tbchat_agente`, `ChatAuditoria` →
`tbchat_auditoria`, `ChatWorkspaceFuncao` → `tbchat_workspace_funcao`,
`ChatProvedorConfig` → `tbchat_provedor`, `ConfiguracaoSistema` →
`tbconfiguracao_sistema` (`CFG_Chave` PK, `CFG_Valor`, `CFG_Segredo`,
`CFG_AtualizadoEm`, `CFG_AtualizadoPor`) e `ConhecimentoRevisorFuncao` →
`tbconhecimento_revisor_funcao`. Com o chat no AnythingLLM, elas deixaram de ser
lidas pelos endpoints de conversa. Inventário em
[banco-dados](../arquitetura/banco-dados/INDEX.md).

## 6. Serviços e lógica

- backend: `ChatSsoController` (SSO), `ConfiguracaoSistemaService` (chaves de
  `tbconfiguracao_sistema`), `ChaveMestraProvider`/`ChaveMestraInicializadora`,
  `GiteaTokenService`, `ConhecimentoService` e `ConhecimentoSyncService`.
  O registro está em `Configuration/ConhecimentoServiceCollectionExtensions.cs`,
  cujo comentário registra que o chat/RAG é servido pelo AnythingLLM.
- frontend: `ChatIframeComponent` (busca a URL e embute o iframe) e
  `ConhecimentoService`/`DocumentEditorComponent`/`ConhecimentoAdminComponent`.

## 7. Regras de negócio

- **Identidade do chat**: o usuário do AnythingLLM é localizado pelo `sub` do
  JWT (`username`), com comparação `OrdinalIgnoreCase`. Sem correspondência, o
  operador precisa de conta criada pelo administrador (`404`).
- **Configuração**: `AnythingLLM:BaseUrl` e `AnythingLLM:ApiKey` são
  obrigatórios (`502` se ausentes); `AnythingLLM:InstanceUrl` define o host do
  iframe e, sem ele, é derivado de `BaseUrl` removendo `/api/v1`;
  `AnythingLLM:TimeoutSeconds` (default 30).
- **Token**: o token de login é temporário e devolvido ao front; não é logado.
- **Chave-mestra**: na primeira subida, se `Acessos:ChaveCifra` não for base64
  válido de 32 bytes, a chave é gerada e gravada em `tbconfiguracao_sistema`
  cifrada pelo DataProtection. Se existir e for válida, tem precedência.
- **Token do Gitea**: `gitea.service_token` é gravado cifrado (AES-GCM,
  envelope `v1:iv:tag:ciphertext`), com fallback para `Gitea:ServiceToken`.
- **Staging configurável**: `Conhecimento:PastaRevisao` (default `07-revisao/`);
  `ConhecimentoPathPolicy` aceita `01..06`, staging, `09-financeiro/` e
  `10-comercial/` na leitura; aprovação só em `01..06`, `09-financeiro/` e
  `10-comercial/`. `ResolverDestinoAprovacao` usa `pasta_destino` (vence a
  `pasta_sugerida`); sem nenhuma, `400`.
- **Escopos**: `financeiro` = `01-empresa/ + 09-financeiro/ + 10-comercial/`;
  `comercial` também inclui `10-comercial/`.
- **Sync**: mapa pasta→workspace vem de `_config/workspaces.yml` no Gitea, lido
  por `scripts/knowledge-sync/sync.py#load_workspace_map`, com fallback para
  `WORKSPACE_ROOTS`. Exclui `07-revisao/`, `08-pendente/`, `BAIXA_QUALIDADE/`,
  `99-historico/`, `_templates/` e `_config/`.
- Conhecimento é lido/gravado por caminho + `baseSha` (concorrência otimista).

## 8. Autorização

`/conhecimento` valida revisor/admin no serviço (`ObterPermissaoAsync`); o
`revisorGuard` é só UX. `/chat` exige `aba.inicio.agentes`; o backend é a fonte
da verdade.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) |
|---|---|---|
| "chat embutido / login no AnythingLLM" | `backend/Central_BackEnd/Controllers/ChatSsoController.cs#Sso` | `frontend/src/app/features/chat/chat-iframe.component.ts#ChatIframeComponent` |
| "salvar token do Gitea / chave-mestra" | `backend/Central_BackEnd/Services/Conhecimento/GiteaTokenService.cs`, `backend/Central_BackEnd/Services/Configuracao/ConfiguracaoSistemaService.cs`, `backend/Central_BackEnd/Services/Configuracao/ChaveMestraProvider.cs` | `frontend/src/app/features/conhecimento/conhecimento-admin.component.ts#ConhecimentoAdminComponent` |
| "origem da URL da API" | — | `frontend/src/app/core/services/config.service.ts#ConfigService` |
| "revisar documentos" | `backend/Central_BackEnd/Services/Conhecimento/ConhecimentoService.cs#SalvarAsync` | `frontend/src/app/features/conhecimento/document-editor.component.ts#DocumentEditorComponent` |
| "pasta de revisão/destino" | `backend/Central_BackEnd/Services/Conhecimento/ConhecimentoPathPolicy.cs#ResolverDestinoAprovacao` | `document-editor.component.ts` (modal de aprovação) |
| "sincronizar base" | `scripts/knowledge-sync/sync.py#sync` | — |
| "configurar ambiente" | `web.config` do servidor (`AnythingLLM__*`, `Acessos__ChaveCifra`, `Gitea__ServiceToken`) + `scripts/deploy/deploy-env.ps1` | `runtime-config.json` (`apiBaseUrl`) |

## 10. Contratos que não podem quebrar

- Caminho validado por `ConhecimentoPathPolicy` (sem `..`/absoluto); sem
  `baseSha` não se salva.
- O token do Gitea (OAuth e serviço) nunca volta ao cliente — só máscara e
  origem.
- Aprovação nunca aceita staging como destino; sem pasta, `400`.
- O chat não pode transformar falha do AnythingLLM em `500`: usa `502`/`504`.

## 11. Armadilhas conhecidas

- **URL relativa à raiz resolve no FRONT**: `/api/chat/*` e
  `/api/conhecimento/*` relativos caem no origin do front (`:1010`), que devolve
  o `index.html` do SPA. Use `apiRootUrl`.
- **Condição inválida no `web.config` do front derruba os dois sites**: uma
  condição `RegexPattern` no fallback SPA fez o URL Rewrite não inicializar e
  tudo responder `500/50/13`. Foi revertida (`a5e2df6`); não reintroduzir sem
  validar no servidor.
- **Chaveiro do DataProtection**: perder o diretório torna a chave-mestra
  ilegível; fazer backup.
- Sem token do Gitea em nenhuma origem, aprovar/rejeitar responde `503`.
- Diagnóstico do sync:
  [troubleshooting/12-11](../operacao/troubleshooting/12-11-chat-e-conhecimento.md).

## 12. Testes

Backend: `ConhecimentoPathPolicyTests`, `ConfiguracaoSistemaTests` (5),
`GiteaTokenServiceTests` (5), `ConhecimentoIntegracoesControllerTests` (5).
Frontend: `conhecimento-admin.component.spec.ts`. Sync: `test_sync.py` (13/13).
Não há teste automatizado do `ChatSsoController` nem do iframe.
QA: `docs/referencia/qa/12-7-chat-e-conhecimento.md`.

## 13. Orçamento de leitura

1 ficha, 4 trechos de código e ≤ 12 tool calls; depois só o arquivo da seção 9
e o teste.

## 14. Pendências

- **Chat**: a conversa roda no AnythingLLM; a Central só faz o SSO. Os endpoints
  antigos de agentes/pergunta e os serviços `ChatPerguntarService`,
  `ChatAgenteCatalogo`, `ChatAgenteStatusService` foram removidos em `cb36ad3`.
- **Chaveiro do DataProtection**: perder o diretório torna a chave-mestra
  ilegível; fazer backup.
- Sem token do Gitea em nenhuma origem, aprovar/rejeitar responde `503`.
- Histórico de fases e corpus: `docs/planos/chat-conhecimento/README.md`
  (planos antigos em `_arquivo/`).
