---
id: sec.12-11-recursos-removidos-e-codigo-dormente
titulo: "11. Recursos removidos e código dormente"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas, componentes]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 610
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 11. Recursos removidos e código dormente

### 11.1 Etapas Globais removidas

No working tree atual não existem componentes, serviços, tipos ou rotas administrativas de Etapas Globais. Também não foi localizada chamada ao endpoint global `/implantacao/etapas`.

Os únicos componentes de etapa do frontend são `ProjetoEtapaCardComponent` e `ProjetoEtapaModalComponent`. Eles trabalham com cards por Projeto e não devem ser confundidos com uma coleção global de etapas.

### 11.2 Outros itens sem uso roteado

| Item | Situação |
|---|---|
| `AdminDashboardService` / `AdminDashboardController` (**backend**) | **Permanecem sem tela**: o endpoint `GET /api/v1/admin/dashboard` existe e está registrado no DI, mas o `AdminDashboardComponent` e o `AdminDashboardService` do frontend — que o consumiam — foram removidos em 2026-10-20 (Fase 0) |
| ramo "modo legado" do `KanbanComponent` | Código dormente: desde 2026-10-20 nenhuma rota carrega o componente sem `data.modo = 'geral'` |
| rota comentada `/administrativo` | Não registrada; o único caminho do quadro é `/kanban` |
| `environment.useMockAuth` | Campo existe como `false` nos dois ambientes, mas não é consultado pelo código |
| mensagens e sessão do widget | Contexto de tarefa e suas últimas 30 mensagens usam `sessionStorage`; não existe endpoint de histórico de conversa |
| `filtrarMinhasConversas` (`whatsapp.utils.ts`) | **Removido em 2026-10-30** (Fase 9b do plano de evolução WhatsApp): a busca de Minhas Conversas passou a ser **server-side** (param `busca` de `GET /atendimentos` com debounce), e a helper local ficou sem uso; o bloco de spec correspondente saiu de `whatsapp.utils.spec.ts` |

Removidos na Fase 0 (2026-10-20) e **não** mais órfãos: `features/historias-telas/historias-telas.component.*` (só o arquivo de dados permanece), `features/admin/pages/dashboard/` e `features/admin/services/` (frontend) e `features/perfil/perfil.routes.ts` (nunca importado — a rota `/perfil` é declarada direto em `features.routes.ts`). `features/chat/` e `features/conhecimento/` são ativos.

