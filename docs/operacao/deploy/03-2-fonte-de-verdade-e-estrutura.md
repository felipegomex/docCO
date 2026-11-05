---
id: sec.03-2-fonte-de-verdade-e-estrutura
titulo: "2. Fonte de verdade e estrutura"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 270
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 2. Fonte de verdade e estrutura

### Monorepo

- Frontend Angular 18: `frontend/`; backend ASP.NET Core 8: `backend/Central_BackEnd/`.
- Gate: `scripts/validate.ps1`; package/publicação: `scripts/deploy/deploy.ps1`; smoke: `scripts/smoke.ps1`.
- Migrations: `backend/Central_BackEnd/Migrations/`; saída local ignorada pelo Git: `deploy/`.

Não existem submódulos. `frontend/` e `backend/` pertencem ao mesmo repositório.

### Saídas reais

Frontend: `frontend/dist/central-conhecimento-actyon/{browser,server}` (IIS recebe somente `browser`); backend: `backend/Central_BackEnd/publish_output/{Central_BackEnd.dll,Central_BackEnd.runtimeconfig.json}`; pacote: `deploy/{BUILD_INFO.txt,backend,frontend}`.

