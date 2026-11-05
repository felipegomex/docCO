---
id: estrutura-projeto.02
titulo: "2. O monorepo real"
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
tokens_estimados: 691
ficha: docs/modulos/kanban.md
---
> Parte de [00 — Estrutura, leitura e fluxo de trabalho](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## 2. O monorepo real

O repositório reúne aplicação, documentação e operação no mesmo projeto Git:

```text
/
├── frontend/                    # uma aplicação Angular
│   ├── angular.json
│   ├── package.json
│   ├── prerender-routes.txt   # lista de rotas; sem prerender ativo
│   ├── public/
│   │   ├── favicon.ico
│   │   └── web.config           # fallback SPA do IIS
│   └── src/
│       ├── main.ts
│       ├── environments/
│       └── app/
│           ├── core/           # auth, guards, interceptor e serviços globais
│           ├── features/       # módulos de produto
│           ├── layout/         # shell principal
│           └── shared/         # componentes, configuração e utilitários
├── backend/
│   ├── Central_BackEnd/        # uma API ASP.NET Core
│   │   ├── Program.cs
│   │   ├── Controllers/
│   │   ├── Hubs/                # hub SignalR do WhatsApp (/hubs/whatsapp)
│   │   ├── Services/
│   │   ├── Models/
│   │   ├── Data/
│   │   ├── Dtos/
│   │   ├── Migrations/
│   │   ├── wwwroot/
│   │   ├── Properties/
│   │   └── appsettings*.json
│   └── Central_BackEnd.Tests/  # único projeto de teste do repositório (xunit)
├── docs/                       # documentação canônica e planos
│   ├── arquitetura/            # estrutura e leitura, visão geral, componentes, banco de dados
│   ├── referencia/             # regras de negócio, endpoints, frontend, backend, QA de histórias
│   ├── operacao/               # deploy e troubleshooting
│   ├── planos/                 # planos de trabalho em execução
│   └── ProjetoFeatures/        # material local não versionado (.gitignore)
├── material-de-projeto/        # material fora do canônico, movido de docs/ na Fase 4b (2026-10-18)
│   ├── backup/                 # auditoria antiga do backend e plano de migração — cópias imutáveis
│   └── ideias/                 # projeto em construção — fora do conjunto canônico
├── scripts/                    # validação, smoke, deploy e extração
├── .opencode/
│   ├── agents/                  # agents do opencode
│   └── skills/                  # skills operacionais do opencode
├── .agents/
│   └── skills/
│       └── frontend-design/     # skill de design frontend
└── AGENTS.md                   # regras automáticas do projeto
```

O caminho real do backend é `backend/Central_BackEnd/`. Não use `backend/src/Central_BackEnd/`: esse caminho aparece em textos antigos e não corresponde ao working tree atual.

