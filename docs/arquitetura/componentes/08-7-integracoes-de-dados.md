---
id: componentes.08
titulo: "7. Integrações de dados"
tipo: arquitetura
area: arquitetura
sistema: central
tags: [arquitetura, componentes, jwt, cors]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 1377
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 7. Integrações de dados

### SQL Server operacional

O `AppDbContext` cobre operadores, funções, tokens de refresh, as duas auditorias, o módulo Acessos Empresas (cliente/unidade e os 5 tipos de acesso; os grupos vêm da legada `tbcliente_grupo`), clientes legados, projetos, etapas, tarefas, agenda, dashboard e Lead/Contato. Em Development, `Database:UseSqlServer=true` seleciona a connection string de homologação; `false` seleciona EF Core InMemory. Fora de Development, o provider é SQL Server.

### Database Explorer

O módulo Database possui uma conexão separada, lida por `DatabaseConnectionService`. A precedência é:

1. variáveis `DB_EXPLORER_*`;
2. seção `DatabaseExplorer` da configuração;
3. configuração vazia; nesse caso, o serviço informa que a conexão não está configurada.

Os endpoints atuais cobrem status, informação, tabelas, colunas, índices, relacionamentos, grafo, busca de colunas, stored procedures, gatilhos, dependências, análise de procedure, construção de consulta, comparação de schemas e comparação de corpos de procedures. Eles não recebem uma instrução SQL arbitrária para execução.

### Chat e Conhecimento

O chat é servido por iframe para o AnythingLLM, autenticado por `GET /api/chat/sso` (`ChatSsoController`). O contrato anterior (`GET /api/chat/agentes`, `POST /api/chat/perguntar`, catálogo `tbchat_agente`, proxy `/api/rag-proxy`) foi removido no commit `cb36ad3` (2026-11-03). As tabelas `tbchat_agente`, `tbchat_auditoria`, `tbchat_workspace_funcao` e a migration `ChatAgentes` permanecem no banco sem uso pelo contrato vigente (ver `banco-dados/INDEX.md`); a configuração de workspaces e revisores segue no fluxo de Conhecimento descrito abaixo.

O Conhecimento (`/api/conhecimento/*`) integra a Central ao repositório Gitea. A autorização é validada no backend por função/revisor ou perfil admin. Cada revisor conecta sua conta Gitea por OAuth pessoal; os tokens são cifrados antes de persistir. O token de serviço prepara branch/alterações/PR tanto para aprovação quanto para rejeição; em ambos os fluxos o revisor aprova e mescla com seu OAuth pessoal. A rejeição conserva o caminho e grava `status: rejeitado` e motivo. Antes de usá-la, `_config/status-documento.yml` no Gitea deve aceitar esse status. Conflito `409` devolve SHA/conteúdo atuais e o editor oferece comparação ou recarga. Auditoria local registra ação/documento/PR/commit, não o conteúdo integral.

O serviço de sincronização em `scripts/knowledge-sync/` lê a branch principal do Gitea, exclui arquivos `INDEX.md` e symlinks, filtra Markdown elegível e atualiza workspaces AnythingLLM; envia `docSource` e mantém manifesto SHA-256 com checkpoints parciais. O roteamento pasta→workspace vem de `_config/workspaces.yml` no repositório Gitea (editável por PR), com fallback para o mapa embutido do `sync.py`: `suporte` recebe `04/02/03/05`, `implantacao`/`desenvolvimento`/`ciaa` recebem `04/02/03`, `financeiro` recebe `09-financeiro/` e `10-comercial/` e `glossario` recebe `06-glossario/`; `07-revisao/` é excluído (staging). A API pode criar workspaces ausentes no primeiro sync se a instalação suportar, mas isso não cria vínculos de revisores (o SQL já semeia `tbchat_workspace_funcao` para as funções 1, 2, 3 e 5). Na VM, o `jca-sync-trigger.service` expõe `POST /api/sync` e `GET /api/status` na porta **8090** com `X-Central-Sync-Key`; a Central dispara por `POST /api/conhecimento/sincronizar` e lê o estado em `GET /api/conhecimento/sincronizacao` (campo `KnowledgeSync__TriggerUrl`/`KnowledgeSync__Secret`).

As chamadas Angular usam URL relativa. O `web.config` versionado contém fallback SPA, não regra de proxy para `/api`; encaminhamento IIS/reverse proxy entre origens não está comprovado no repositório. A disponibilidade exige configuração efetiva de AnythingLLM, Claude, OAuth/ServiceToken Gitea, VM e mapeamentos de workspace.

