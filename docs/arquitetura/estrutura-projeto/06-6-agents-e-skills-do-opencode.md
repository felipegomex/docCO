---
id: estrutura-projeto.06
titulo: "6. Agents e skills do opencode"
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
tokens_estimados: 684
ficha: docs/modulos/kanban.md
---
> Parte de [00 — Estrutura, leitura e fluxo de trabalho](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## 6. Agents e skills do opencode

Os agents estão em `.opencode/agents/`. As skills operacionais do opencode estão em `.opencode/skills/`; a skill `frontend-design` é uma definição separada em `.agents/skills/frontend-design/`. `AGENTS.md` define quando a delegação automática deve ocorrer.

### Agents

| Agent | Responsabilidade |
|---|---|
| `docs-writer` | Sincronizar Markdown com o código, sem alterar aplicação |
| `content-editor` | Editar conteúdo do wiki em `*.data.ts` |
| `angular-engineer` | Alterações Angular e aplicação da skill de design |
| `dotnet-engineer` | Alterações ASP.NET Core, EF Core, JWT e controllers |
| `security-auditor` | Revisão somente leitura de segurança |

### Skills

| Skill | Localização | Uso |
|---|---|---|
| `central-dev-setup` | `.opencode/skills/central-dev-setup/` | Ambiente de desenvolvimento com reload automático: `devsetup.ps1`, `Makefile`, tasks do VS Code, `launch.json`, guia de 5 minutos e troubleshooting |
| `subir-interno` | `.opencode/skills/subir-interno/` | Atalho para subir frontend e backend em desenvolvimento; delega o HOW para `central-dev-setup` |
| `validar` | `.opencode/skills/validar/` | Executar o gate de build por `scripts/validate.ps1` |
| `deploy-limpo` | `.opencode/skills/deploy-limpo/` | Orquestar validação, empacotamento, publicação e smoke |
| `frontend-design` | `.agents/skills/frontend-design/` | Diretrizes para criação e reformulação visual |

O `Makefile` da raiz do repositório pertence à skill `central-dev-setup`, mas fica na raiz porque o `make` resolve `FRONTEND_DIR` e `BACKEND_DIR` a partir do diretório de onde é invocado. Ele **não** é o gate de build: o gate oficial é `scripts/validate.ps1`.

### Utilitários de `scripts/` que não são gate

| Script | O que faz | Observação |
|---|---|---|
| `scripts/analyze-components.ps1` | Auditoria estática de otimização dos componentes Angular (tamanho, `ChangeDetectionStrategy`, `trackBy`, subscrições) | **Somente leitura**: não altera código. O relatório vai para `%TEMP%\otimizacao-componentes.md` (descartável, não versionado). Não roda no CI e não é parte do gate — é ferramenta de manutenção pontual |

O gate de build é `scripts/validate.ps1`; o smoke pós-deploy é `scripts/smoke.ps1`; a publicação é `scripts/deploy/deploy.ps1`.

Agentes não ampliam o escopo automático: uma tarefa de código continua sujeita às permissões e aos limites explícitos do pedido.

