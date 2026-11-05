---
id: sec.19-regressao-historico-funcionalidades-desativadas-ou-sem-rota-ativa
titulo: "Regressão/histórico — funcionalidades desativadas ou sem rota ativa"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 718
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# Regressão/histórico — funcionalidades desativadas ou sem rota ativa

Estes itens foram removidos do fluxo ou mantidos apenas como compatibilidade. **Não contam como histórias ativas** e não devem ser testados como se fossem telas disponíveis.

| Item | Evidência na árvore de trabalho | Tratamento |
|---|---|---|
| Rota antiga `/api/rag-proxy` (e o contrato `/api/chat/agentes`/`/api/chat/perguntar`) | Substituídos por `GET /api/chat/sso` + iframe do AnythingLLM; não são endpoints ativos desde o `cb36ad3`. | histórico; usar CJ-01. |
| Central Executiva `/executivo` | Pasta/rota não existem no código atual. | desativada; não criar link ativo. |
| Módulo Gestor `/gestor/entrada` | Backend e frontend do módulo não estão na árvore de trabalho. | desativado; não é destino pós-login. |
| Kanban ADM legado `/administrativo` | Rota está comentada em `features.routes.ts`; o item de menu saiu em 2026-10-20 (fase 0) por duplicar `/kanban`. | desativado como caminho; AK-02 trata o link direto ativo. |
| Gestão da Central `/admin/dashboard` | Componente, serviço e controller existem, mas a rota não está em `admin.routes.ts` nem no menu. | sem rota ativa; não documentar como tela disponível. |
| Etapas Globais | Rotas, componentes, serviço e API global foram removidos; a arquitetura atual usa nove `tbprojetoetapa` por projeto. | histórico; usar IM-02/IM-04/IM-06. |
| Database Diagrama | Componente e rota não existem; a navegação atual tem seis abas. | removido; não prometer diagrama. |
| Database IA Chat | Componente/rota removidos; o JOTA global é separado. | removido; não tratar como IA do Database. |
| Relatório separado de Implantação | A rota redireciona para o Dashboard e o cálculo está na seção Produtividade. | redirecionamento em IM-01, não tela independente. |
| Home “Continue de onde parou”/“Mais utilizados” | Blocos não estão no template atual. | não são história ativa. |
| Acessos sobre planilha do Google Sheets | `Services/GoogleSheetsService.cs`, `IGoogleSheetsService`, `Models/Acessos/{EmpresaAcesso,EmpresaResumo,EmpresaDetalheResponse}.cs`, a seção `GoogleSheets` do `appsettings`, `features/acessos/acessos.component.*` e `features/services/acessos.service.ts` foram removidos. Os 3 endpoints antigos (`GET /acessos`, `POST /acessos/validar-senha`, `POST /acessos/visualizar`) não existem mais — o gate hoje é o `POST /acessos/clientes/{id}/autorizar`, que emite ticket em vez de marcar "validado". | histórico; use CF-04/CF-12. O endereço `/ferramentas/acessos` redireciona para `/acessos/clientes`. |

