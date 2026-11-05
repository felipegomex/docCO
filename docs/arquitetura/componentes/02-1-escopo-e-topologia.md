---
id: componentes.02
titulo: "1. Escopo e topologia"
tipo: arquitetura
area: arquitetura
sistema: central
tags: [arquitetura, componentes, jwt, cors]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 404
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 1. Escopo e topologia

A implantação atual possui:

- **uma aplicação frontend Angular 18**, standalone, somente browser (CSR);
- **um backend ASP.NET Core 8**, expondo APIs REST versionadas, endpoints Chat/Conhecimento sem versão e o **hub SignalR `/hubs/whatsapp`** (notificação de saída do módulo WhatsApp);
- SQL Server como persistência principal, incluindo o módulo de Acessos Empresas, e fonte do Database Explorer;
- AnythingLLM e Claude para recuperação/geração do Chat; Gitea como repositório de conhecimento, com sincronizador externo em VM.

Fluxo de alto nível:

```text
Navegador
   |
   | HTTP :1010
   v
IIS — saída browser do Angular e fallback SPA
   |
   | HTTP :1009, Bearer + cookie de refresh
   v
ASP.NET Core — um único backend
   |----> /hubs/whatsapp (SignalR, notificação de saída do WhatsApp)
   |----> SQL Server operacional (EF Core, inclusive Acessos Empresas)
   |----> SQL Server do Database Explorer (configuração própria)
   |----> AnythingLLM (recuperação por workspace) + Claude (resposta)
   |----> Gitea (documentos, branches e pull requests)
   `----> VM de sincronização (Gitea → AnythingLLM; timer e gatilho interno)
```

Monorepo único: uma SPA Angular + uma Web API. A divisão em quatro webapps nunca foi implementada (plano removido em 09/10).

