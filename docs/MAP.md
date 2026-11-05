---
titulo: "Mapa de contexto da Central de Operação"
status: "gerado"
revisado_em: "2026-10-28"
---

# Mapa de contexto

> **GERADO** por `scripts/gen-docs-map.ps1` a partir de `docs/index.yaml`. Não edite à mão.
> Leia esta pãgina primeiro, depois **uma** ficha de módulo e só então o documento profundo.
> Stubs (`status: dividido`) não são fonte: abra o `INDEX.md` da subpasta.

## Rotas por intenção

| Intenção | Ficha / entrada | Documentos profundos |
|---|---|---|
| comecar, indice, navegacao, documentacao | [README](./README.md) | [glossãrio](./glossario.md) |
| acessos, cliente, unidade, senha, rdp, vpn, anydesk, banco, actyon | [Acessos Empresas](./modulos/acessos.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA Acessos](./referencia/qa/14-9-acessos-empresas.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| kanban, tarefa, projeto, coluna, funcao, bloqueio, gargalo, implantacao | [Kanban Geral](./modulos/kanban.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA Kanban](./referencia/qa/06-3-1-kanban-geral.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| dashboard, kpi, comercial, lead, proposta, administrativo, financeiro, custo, orcamento, contrato | [Dashboard Integrado](./modulos/dashboard.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| login, jwt, cookie, refresh, 401, 403, permissao, rbac, cargo, operador, auditoria, sistema | [Auth, RBAC e Sistema](./modulos/auth-rbac-sistema.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| database, explorador, tabela, coluna, schema, procedure, trigger, comparacao, script | [Database Explorer](./modulos/database-explorer.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| agenda, evento, compromisso, recorrencia, serie, conflito, visibilidade | [Agenda](./modulos/agenda.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| chat, rag, agente, conhecimento, documento, revisao, gitea, tbdoc | [Chat e Conhecimento](./modulos/chat-conhecimento.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| ideias, melhoria, erro, sugestao, aprovacao, especificacao | [Ideias, Melhorias e Erros](./modulos/ideias.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| whatsapp, atendimento, conversa, fila, numero, qr, webhook, sla, metrica, etiqueta, retencao | [Atendimento WhatsApp](./modulos/whatsapp.md) | [backend](./referencia/backend/INDEX.md) | [frontend](./referencia/frontend/INDEX.md) | [QA](./referencia/qa/INDEX.md) | [dados](./arquitetura/banco-dados/INDEX.md) |
| estrutura, monorepo, convencoes, segredos, pipeline, agents, skills | [estrutura-projeto](./arquitetura/estrutura-projeto/INDEX.md) | [seções](./arquitetura/estrutura-projeto/INDEX.md) |
| visao geral, modulos, rotas, ambientes, roadmap | [visao-geral](./arquitetura/visao-geral/INDEX.md) | [seções](./arquitetura/visao-geral/INDEX.md) |
| arquitetura, componentes, jwt, cors, iis, chat, rag, anythingllm | [componentes](./arquitetura/componentes/INDEX.md) | [seções](./arquitetura/componentes/INDEX.md) |
| tabela, coluna, schema, migration, entidade, banco, sqlserver | [banco-dados](./arquitetura/banco-dados/INDEX.md) | [seções](./arquitetura/banco-dados/INDEX.md) |
| regra de negocio, permissao, rbac, faturamento, contrato, kanban, acessos, whatsapp, chat | [regras-negocio](./referencia/regras-negocio/INDEX.md) | [seções](./referencia/regras-negocio/INDEX.md) |
| endpoint, api, rota, contrato, autorizacao, http | [endpoints](./referencia/endpoints.md) | [seções](./referencia/endpoints.md) |
| whatsapp, implementacao, servicos, contratos, testes, operacao | [whatsapp-referencia](./referencia/whatsapp/INDEX.md) | [seções](./referencia/whatsapp/INDEX.md) |
| tela, componente, frontend, angular, rota, guard, service, scss | [frontend](./referencia/frontend/INDEX.md) | [seções](./referencia/frontend/INDEX.md) |
| backend, controller, service, di, efcore, jwt, csharp | [backend](./referencia/backend/INDEX.md) | [seções](./referencia/backend/INDEX.md) |
| qa, historia, cenario, teste, tela, validacao | [qa-historias](./referencia/qa/INDEX.md) | [seções](./referencia/qa/INDEX.md) |
| erro, falha, 500, 401, cors, iis, login, migration, whatsapp | [troubleshooting](./operacao/troubleshooting/INDEX.md) | [seções](./operacao/troubleshooting/INDEX.md) |
| deploy, publicar, iis, migration, backup, rollback, smoke, producao | [deploy](./operacao/deploy/INDEX.md) | [seções](./operacao/deploy/INDEX.md) |
| setup, ambiente local, dotnet watch, ng serve, porta ocupada, cors local | [setup-local](./operacao/setup-local.md) | - |
| iis, servidor, publicacao, webconfig | [iis-setup](./operacao/iis-setup.md) | - |
| auditoria, seguranca, riscos, revisao | [auditoria-seguranca](./operacao/auditoria-segurança.md) | - |
| openwa, gateway, whatsapp, docker, webhook | [instalacao-openwa](./operacao/instalacao-openwa/INDEX.md) | [seções](./operacao/instalacao-openwa/INDEX.md) |
| termo, glossario, significado | [glossãrio](./glossario.md) | - |

## Orçamento de contexto

- Abra no mãximo **1 ficha** e **4 trechos de código** por tarefa.
- Comece pela ficha; só abra a seçõo do documento profundo que ela citar.
- Não abra stub (`status: dividido`); use o `INDEX.md` da subpasta.
- Se a ficha e o código divergirem, pare e pergunte antes de alterar.
