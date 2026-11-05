---
id: visao-geral.09
titulo: "7. Roadmap: atual, plano e histórico"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 2053
ficha: docs/modulos/README.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 7. Roadmap: atual, plano e histórico

| Situação | Item | Como interpretar |
|---|---|---|
| Atual | Um frontend Angular e um backend ASP.NET Core | Arquitetura vigente e único bundle/site frontend |
| Implementado no código | Autenticação, Agenda, Dashboard Integrado, Acessos Empresas, Implantação, Database Explorer, Chat e revisão de Conhecimento | Componentes/rotas e serviços existem; o Chat está no ar (iframe + SSO) |
| Disponibilizado em 2026-10 | SQL do Chat/Conhecimento | SQL idempotente (ChatConhecimento e migrations de chat) aplicado em HML e PROD em 2026-10-30; segredos/VM configurados no servidor |
| Atual | Chat com AnythingLLM + Claude e sincronização Gitea → AnythingLLM | Integrações implementadas no código/script; publicadas em HML (.130) e PROD (.254) em 2026-10-28; SQL aplicado em 2026-10-30 |
| Atual | Módulo Acessos Empresas em 6 tabelas `tb*` geridas + legadas `tbcliente_grupo` (grupos) e `tbcliente` (empresa do ERP) | Unidade → 5 tipos de acesso, com grupos legados e criação de unidade a partir do ERP; **substituiu** a leitura do Google Sheets |
| Atual | **Kanban Geral** por função (`/kanban`, `/kanban/gargalos`, `/kanban/configurar`, `/kanban/admin/colunas`) | Quadro único de tarefas e projetos por função (`tbfuncao`); `FuncaoAuthorizationService` decide leitura/escrita por cargo `GESTOR_MASTER` + `tbgestor`; bloqueio é atributo do card, não coluna; **catálogo de colunas compartilhado** + **visão por setor** (`tbcolunafuncao`: visibilidade e máscara de nome), funções e etapas da função somente-leitura na tela |
| Atual | **RBAC e gestão de operadores** (`/sistema/funcoes-administrativas`, `/sistema/operadores`, `/sistema/permissoes`) | Cargo (`tbcargo`) separado do setor (`FUNCAO_ID`), permissões por recurso/ação/escopo (`tbperm_cargo`, `tbperm_operador`), policies `perm:<recurso>:<acao>`, auditoria de 403 e telas de operadores/permissões. Schema aplicado em `dbBUSINESS_HML` em 2026-10-21 (4 tabelas + `CARGO_ID`; 10 cargos) por scripts idempotentes, com preflight bloqueante no deploy. Em 2026-10-27, `CARGO_Nivel`/`CARGO_FuncaoId` e `TECH_LEAD_ADM` (11 cargos) passaram a limitar a edição da matriz (Master em tudo, exceto o próprio Master; Tech Lead no mesmo setor e nível inferior) |
| Atual | **Navegação por setor e visões gerais** (`/suporte/visao-geral`, `/desenvolvimento/visao-geral`, `/ciaa/visao-geral`, `/ciaa/projetos`, `/base-conhecimento`) | Sidebar reorganizada em 2026-10-22; dashboard e projetos reutilizados com `data.funcaoId`; Base de Conhecimento consolida Stack, Glossário e Conhecimento Rápido. Plano em [`planos/rbac-fechamento.md`](../../planos/rbac-fechamento.md) |
| Atual | Primeiro projeto de teste do repositório (`backend/Central_BackEnd.Tests`) | xunit + FluentAssertions; `scripts/validate.ps1` executa a suíte no passo **5/7** (o inventário por arquivo está em [`referencia/backend.md`](../../referencia/backend.md) § 5.5). |
| Atual | **Módulo de atendimento WhatsApp** (`/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/metricas`, `/whatsapp/numeros`, `/whatsapp/minhas-conversas`, `/whatsapp/auditoria`, `/whatsapp/ia-configuracao`, `/whatsapp/respostas-rapidas`, `/whatsapp/configuracao`, `/whatsapp/sla`, `/whatsapp/retencao`, `/whatsapp/admin`, `/whatsapp/admin/contatos`, `/whatsapp/etiquetas`) | **Implementado no código (Fases A, C, E, F, G)**: fila em Kanban (4 colunas), painel do atendimento, atribuição pela primeira resposta, transferência, devolução, resolução com mensagem de encerramento, reabertura, estado "Aguardando cliente", nota interna, **métricas de qualidade: taxa reabertura, FCR, transferências, por cliente (Fase E)**, **auditoria genérica append-only (Fase F)**, **IA: configuração admin (Gemini) + assistente por atendimento com resumo/classificação/sugestões (Fase G)**, métricas operacionais, administração de números, respostas rápidas, SLA, retenção de mídia. **Fase 08**: hub administrativo (`/whatsapp/admin`) com indicadores lidos de endpoints existentes e atalhos, e **gestão de contatos** (`/whatsapp/admin/contatos`) com filtros server-side, edição pelo PATCH da Fase 01 (409 recarrega a ficha), histórico em painel e aviso de `@lid`; o catálogo `/whatsapp/etiquetas` passa a ser acessível pelo hub. **Fase 09**: atualização em tempo real pelo hub SignalR `/hubs/whatsapp` (eventos de atendimento, mensagem, ack, evento, fila e digitação), reconexão automática e **indicador de digitação** interno/externo; o polling de 30 s permanece como fallback e **não há schema novo**. **Fase D (categorias/tags) foi removida** em 2026-10-23 (decisão de correção total). **Novo ciclo Service Desk (Fases 01–10, concluído — [documentação única do módulo](../../../material-de-projeto/atendimento-whatsapp/README.md))**: vínculo do contato com a empresa do ERP (ficha, typeahead, PATCH com lock otimista por rowversion — 409 `CONFLICT_CONTATO_VERSAO`), layout do atendimento em **3 colunas com cabeçalho fixo** (colunas recolhíveis persistidas), contexto do cliente carregado junto ao atendimento, compositor com **seletor de destino** (mensagem × nota interna), SLA com situação **`critico`** a partir do percentual de aviso configurável (50–99), **Fase 06** com **etiquetas** (catálogo admin + aplicação pelo atendente), **investigação** (atributo booleano, não status) e **observação interna** do atendimento na aba Resumo, e **Fase 07** com o **Kanban Service Desk**: card próprio (SLA destacado, empresa ou “Cliente pendente”, assunto, até 4 etiquetas + contador, badge de investigação, responsável/tempo e ações por status/permissão), filtros de prioridade, empresa, etiqueta, investigação e as duas dimensões do SLA aplicados **no servidor** (o total/paginação acompanham o recorte) e apenas a ordem local à página; no mobile o quadro rola na horizontal. A Fase 07 **não altera o schema**. Persistência em 21 tabelas `tbwhatsapp_*` (script próprio, **aplicado em `dbBUSINESS_HML` e `dbCENTRAL_OPERACAO`**) e gateway **OpenWA v0.25.0** **no ar** na VM Ubuntu `192.168.2.253`. O módulo nasce **desabilitado** no `appsettings.json` versionado e é ligado por variável de ambiente no servidor. **HML e PROD publicados em 2026-10-26**: HML com smoke 5/5 e PROD com preflight e smoke 5/5; a matriz visual manual segue pendente. Documentação única em [`material-de-projeto/atendimento-whatsapp/README.md`](../../../material-de-projeto/atendimento-whatsapp/README.md) |
| Atual | Deploy IIS, backup, preservação de `appsettings*.json`, `BUILD_INFO` e smoke | Fluxo operacional documentado em [`10-DEPLOY.md`](../../operacao/deploy.md) |
| Plano | Migração de um para quatro webapps | Desenho **removido** em 09/10 (nunca implementado; histórico no Git) |
| Atual | Reorganização de documentos legados em backup | Executada; snapshot `docs/backup/2026-09-24/` **removido** em 09/10 após consolidação (histórico no Git) |
| Histórico | Módulo Gestor e Central Executiva removidos | Não tratar `/executivo` ou componentes antigos como funcionalidade atual |
| Histórico | `/api/rag-proxy`, `/api/chat/agentes` e `/api/chat/perguntar` | Substituídos por `GET /api/chat/sso` e página `/chat` (iframe); removidos no `cb36ad3`; o widget JOTA global não existe mais |
| Histórico | `/administrativo` comentado | A rota não está ativa; o único caminho do quadro é `/kanban` (o atalho de menu "Kanban ADM" saiu em 2026-10-20) |
| Histórico | Acessos lia uma planilha do Google Sheets (3 endpoints, somente leitura, senhas em texto puro) | Substituído pelo módulo CRUD (6 tabelas geridas + legadas; eram 7 — a `tbgrupoacesso` foi dropada); `/ferramentas/acessos` redireciona para `/acessos/clientes` |

A divisão em quatro webapps **não** está implementada e o plano foi removido em 09/10 (histórico no Git). Não usar multi-webapp para descrever o estado atual.
