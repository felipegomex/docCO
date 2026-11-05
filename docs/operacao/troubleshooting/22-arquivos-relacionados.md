---
id: sec.22-arquivos-relacionados
titulo: "Arquivos relacionados"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 855
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# Arquivos relacionados

- [`10-DEPLOY.md`](../deploy.md) â€” pipeline obrigatÃ³rio, migrations, backup e rollback.
- [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) â€” runbook do gateway OpenWA instalado na VM Ubuntu (imagem Docker fixada, override de compose, `.env` na VM, firewall na cadeia `DOCKER-USER`, configuraÃ§Ã£o do backend no IIS, webhook, schema, backup/restore e prova tÃ©cnica).
- [`02-ARQUITETURA.md`](../../arquitetura/componentes/INDEX.md) â€” arquitetura, autenticaÃ§Ã£o e controles de seguranÃ§a.
- [`04-ESTRUTURA-DADOS.md`](../../arquitetura/banco-dados.md) â€” entidades, tabelas, migrations e integraÃ§Ãµes.
- [`05-ENDPOINTS.md`](../../referencia/endpoints.md) â€” contratos HTTP e autorizaÃ§Ã£o.
- [`07-SERVICES-BACKEND.md`](../../referencia/backend.md) â€” controllers, serviÃ§os e persistÃªncia.
- [`../backend/Central_BackEnd/Program.cs`](../../../backend/Central_BackEnd/Program.cs) â€” provedores, CORS, JWT e Swagger.
- [`../frontend/src/app/features/chat/chat-iframe.component.ts`](../../../frontend/src/app/features/chat/chat-iframe.component.ts) — busca `GET /api/chat/sso` na origem do `runtime-config` (`apiRootUrl`) e embute o AnythingLLM em iframe.
- [`../backend/Central_BackEnd/Controllers/ChatSsoController.cs`](../../../backend/Central_BackEnd/Controllers/ChatSsoController.cs) — SSO passthrough: resolve o usuário no AnythingLLM e emite token de login temporário.
- [`../backend/Central_BackEnd/Controllers/ConhecimentoController.cs`](../../../backend/Central_BackEnd/Controllers/ConhecimentoController.cs) â€” endpoints de revisÃ£o, OAuth e sincronizaÃ§Ã£o.
- [`../scripts/knowledge-sync/README.md`](../../../scripts/knowledge-sync/README.md) â€” provisionamento externo e job Gitea â†’ AnythingLLM.
- [`../frontend/src/app/core/interceptors/auth.interceptor.ts`](../../../frontend/src/app/core/interceptors/auth.interceptor.ts) â€” Bearer, credenciais e refresh.
- [`../scripts/validate.ps1`](../../../scripts/validate.ps1) â€” gate de commit/deploy (segredos + build + testes).
- [`../scripts/check-secrets.ps1`](../../../scripts/check-secrets.ps1) â€” bloqueia segredo no Ã­ndice do Git.
- [`../scripts/smoke.ps1`](../../../scripts/smoke.ps1) â€” verificaÃ§Ã£o de disponibilidade atual.
- [`../scripts/deploy/deploy.ps1`](../../../scripts/deploy/deploy.ps1) â€” package e publicaÃ§Ã£o.
- [`../backend/Central_BackEnd.Tests/WhatsAppStatusSessaoTests.cs`](../../../backend/Central_BackEnd.Tests/WhatsAppStatusSessaoTests.cs) â€” status canÃ´nico da sessÃ£o e telefone gravado (Â§19.10.1).
- [`../backend/Central_BackEnd.Tests/WhatsAppFilaRecuperacaoTests.cs`](../../../backend/Central_BackEnd.Tests/WhatsAppFilaRecuperacaoTests.cs) â€” recuperaÃ§Ã£o da linha presa em processamento (Â§19.10.2).

