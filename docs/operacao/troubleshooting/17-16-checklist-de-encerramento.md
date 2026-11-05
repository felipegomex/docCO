---
id: sec.17-16-checklist-de-encerramento
titulo: "16. Checklist de encerramento"
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
tokens_estimados: 565
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 16. Checklist de encerramento

Antes de considerar o incidente resolvido:

- [ ] `validate.ps1` estÃ¡ verde no commit atual; o package Release foi executado separadamente.
- [ ] Branch e hash Git conferem com `BUILD_INFO.txt`.
- [ ] Frontend e backend sobem nas portas esperadas.
- [ ] Login, access token em memÃ³ria e refresh funcionam; o 401 conhecido de logout foi tratado, nÃ£o ignorado.
- [ ] `cc_refresh` Ã© HttpOnly, nÃ£o aparece no armazenamento e a polÃ­tica `Secure` foi avaliada.
- [ ] CORS usa uma origem autorizada.
- [ ] Schema e `__EFMigrationsHistory` do banco alvo foram validados.
- [ ] Se o commit mexeu no Dashboard Integrado: nenhuma rota duplicada em `Controllers/` vs `Controllers/Dashboard/`, nenhum `Contains` de coleÃ§Ã£o local em `IQueryable` (compat 100) e colunas dos espelhos legados confirmadas no dicionÃ¡rio do banco.
- [ ] Acessos Empresas (`/acessos/clientes`, gate, abas e revelaÃ§Ã£o) foi exercitado quando entrou no fluxo â€” o smoke nÃ£o cobre nenhuma dessas rotas.
- [ ] Database Explorer e Chat/Conhecimento foram testados quando fazem parte do fluxo; migration, provedores e permissÃ£o foram considerados.
- [ ] `appsettings*.json` e `runtime-config.json` do servidor permanecem intactos; ausÃªncia no servidor aborta o publish no pre-flight.
- [ ] `app_offline.htm` foi removido somente apÃ³s mirror Ã­ntegro.
- [ ] Smoke automatizado passou (frontend + Swagger + `runtime-config.json` com a `apiBaseUrl` do ambiente).
- [ ] QA manual cobriu pelo menos uma API autenticada e uma tela de negÃ³cio.
- [ ] A regra de atraso Ã© a mesma em todas as telas (Projetos, Dashboard, Kanban): os nÃºmeros de atrasados conferem para o mesmo conjunto.
- [ ] A publicaÃ§Ã£o usou `Get-Credential`/secret manager (`-SenhaRemota` explÃ­cito; o script aborta sem ele).
- [ ] Nenhum segredo foi adicionado a logs, documentaÃ§Ã£o ou comandos.

