---
id: sec.17-arquivos-relacionados
titulo: "Arquivos relacionados"
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
tokens_estimados: 522
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# Arquivos relacionados

- [`09-TROUBLESHOOTING.md`](../troubleshooting.md) — sintomas, causas e soluções.
- [`../scripts/validate.ps1`](../../../scripts/validate.ps1) — gate obrigatório (segredos + git + build + testes).
- [`../scripts/deploy/deploy.ps1`](../../../scripts/deploy/deploy.ps1) — package, publicação multi-ambiente (`-Ambiente HML|PROD`), trava de PROD, pre-flight, backup e recibos.
- [`../frontend/src/app/features/chat/chat-iframe.component.ts`](../../../frontend/src/app/features/chat/chat-iframe.component.ts) — busca a URL do SSO na origem da runtime-config (`apiRootUrl`) e embute o AnythingLLM em iframe.
- [`../backend/Central_BackEnd/Controllers/ChatSsoController.cs`](../../../backend/Central_BackEnd/Controllers/ChatSsoController.cs) — resolve o usuário no AnythingLLM e emite o token de login (SSO).
- [`../backend/Central_BackEnd/Services/Conhecimento/ConhecimentoService.cs`](../../../backend/Central_BackEnd/Services/Conhecimento/ConhecimentoService.cs) — autorização de escopo e revisão Gitea.
- [`../scripts/knowledge-sync/README.md`](../../../scripts/knowledge-sync/README.md) — configuração da VM e job diário de sincronização.
- [`../scripts/smoke.ps1`](../../../scripts/smoke.ps1) — smoke real e seus limites.
- ~~`../scripts/importar_acessos_xlsx.py`~~ — **removido em 2026-10-11**: tinha a connection string com credencial em texto puro. Ver "Segredos no repositório" em [`09-TROUBLESHOOTING.md`](../troubleshooting.md).
- [`../backend/Central_BackEnd/Program.cs`](../../../backend/Central_BackEnd/Program.cs) — ambientes, CORS e Swagger.
- [`../backend/Central_BackEnd/wwwroot/web.config`](../../../backend/Central_BackEnd/wwwroot/web.config) — host ASP.NET Core no IIS.
- [`../frontend/public/web.config`](../../../frontend/public/web.config) — fallback SPA.
