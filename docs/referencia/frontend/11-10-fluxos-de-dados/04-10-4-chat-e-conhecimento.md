---
id: 11-10-fluxos-de-dados.04
titulo: "10.4 Chat e Conhecimento"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, fluxos]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 1350
ficha: docs/modulos/chat-conhecimento.md
---
> Parte de [10. Fluxos de dados](./INDEX.md). Ficha: [Chat e Conhecimento](../../../modulos/chat-conhecimento.md).

### 10.4 Chat e Conhecimento

```text
ChatIframeComponent (/chat)
  -> GET /api/chat/sso -> { url }
  -> iframe do AnythingLLM (allow="microphone")

ConhecimentoShell / telas de revisão
  -> revisorGuardLazy -> GET /api/conhecimento/permissao
  -> ConhecimentoService -> /api/conhecimento/*
```

A página `/chat` (`ChatIframeComponent`) não troca pergunta/resposta com a API: a sessão acontece dentro do iframe, autenticado pelo SSO (`ChatSsoController`). O componente busca `{ url }` em `GET /api/chat/sso` na origem do `runtime-config` (`ConfigService#apiRootUrl`); sem `url` ou em erro, mostra o estado de carregamento ou a mensagem de erro (`mensagem` do backend ou "Falha ao carregar o chat."). Não há catálogo de agentes, seleção de agente, widget global nem mascote nesta página na versão atual — o histórico da conversa fica no AnythingLLM. O contrato anterior (`GET /api/chat/agentes`, `POST /api/chat/perguntar`, `JotaChatService`, `ChatContextoService`, widget global) foi removido no commit `cb36ad3` (2026-11-03).

O guard da revisão não implementa permissão local: consulta o servidor, exige `pode_acessar` e `eh_revisor` e redireciona à raiz em caso negativo ou falha. O OAuth do Gitea é acionado pela tela e vincula a conta pessoal do revisor. A revisão é persistida pelo backend em Gitea, não no estado Angular.

O editor detecta `409` com SHA divergente, mantém as alterações locais e mostra a versão atual do Gitea; o revisor pode comparar conteúdo local e remoto ou recarregar a versão remota (descartando as alterações locais). A tela `/conhecimento/configuracao` lê `GET /api/conhecimento/admin/config` e grava função→workspace e operador→função revisora por `PUT /api/conhecimento/admin/workspaces` e `/admin/revisores`; a autorização administrativa é revalidada pelo backend.
