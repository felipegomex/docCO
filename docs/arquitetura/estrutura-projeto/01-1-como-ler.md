---
id: estrutura-projeto.01
titulo: "1. Como ler"
tipo: arquitetura
area: processo
sistema: central
tags: [monorepo, convencoes, pipeline, agents]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - AGENTS.md
  - scripts/validate.ps1
testes:
  - docs/
tokens_estimados: 579
ficha: docs/modulos/kanban.md
---
> Parte de [00 — Estrutura, leitura e fluxo de trabalho](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## 1. Como ler

### Entrada por perfil

| Perfil da tarefa | Leitura mínima |
|---|---|
| Primeira visita ao repositório | [`README.md`](../../README.md) → este arquivo → [`01-VISAO-GERAL.md`](../visao-geral/INDEX.md) |
| Alteração de tela ou API | este arquivo → [`06-COMPONENTES-FRONTEND.md`](../../referencia/frontend.md) / [`05-ENDPOINTS.md`](../../referencia/endpoints.md) / [`07-SERVICES-BACKEND.md`](../../referencia/backend.md) → rotas/controllers reais |
| Alteração de JWT, cookie ou acesso | este arquivo → [`02-ARQUITETURA.md`](../componentes/INDEX.md) → [`03-REGRAS-NEGOCIO.md`](../../referencia/regras-negocio/INDEX.md) |
| Deploy ou incidente | este arquivo → [`10-DEPLOY.md`](../../operacao/deploy.md) → [`09-TROUBLESHOOTING.md`](../../operacao/troubleshooting.md) |
| Integração de dados | este arquivo → [`02-ARQUITETURA.md`](../componentes/INDEX.md) → [`04-ESTRUTURA-DADOS.md`](../banco-dados.md) |
| Mudança de código com agentes | este arquivo → seção de agents/skills |

### Ordem de confiança

1. Código e configuração do working tree.
2. Documentos canônicos especializados para o assunto.
3. Documentos canônicos geralistas.
4. Materiais históricos, auditorias e planos.

Uma afirmação em um documento antigo não torna o comportamento atual. Em caso de divergência, confirme símbolo, rota, atributo de autorização e configuração antes de atualizar qualquer texto.

### Busca com `Ctrl+F`

Use termos estáveis, sem depender de numeração de seções:

- `autenticacao`, `JWT`, `cc_refresh`, `access token`, `refresh token`;
- `/api/v1`, `/api/chat/sso`, `CORS`, `IIS`;
- `rotas`, `módulos`, `SQL Server`, `AnythingLLM`;
- `acessos empresas`, `tbcliente_grupo`, `CAO_TemRdp`, `Cifrador`, `REVEAL_SENHA`, `Paginado`;
- `branch`, `pipeline`, `BUILD_INFO`, `agents`, `skills`;
- `SignalR`, `/hubs/whatsapp`, `access_token` (hub do WhatsApp);
- `screens-data`, `documentação canônica`, `backup`.

Os links internes apontam para arquivos completos. Não crie nem mantenha links para âncoras profundas.

