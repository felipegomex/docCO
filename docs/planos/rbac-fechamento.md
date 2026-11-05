---
titulo: "RBAC e navegação por setor — fechamento"
status: "em execucao"
criado_em: "2026-11-05"
validade: "2027-06-30"
---

# RBAC e navegação por setor — fechamento

Documento único do que resta da reestruturação de navegação e permissões. Substitui os
planos `docs/planos/rbac/` (7 fases) e `docs/planos/reestruturacao-navegacao-permissoes/`
(5 arquivos), apagados em 2026-11-05 e preservados no histórico do git.

Decisões: [`ADR 004`](../decisoes/004-rbac.md), [`ADR 005`](../decisoes/005-cifra-aes-ativa.md),
[`ADR 003`](../decisoes/003-sem-staging.md). Ficha: [`auth-rbac-sistema`](../modulos/auth-rbac-sistema.md).

## A regra do dono do projeto

> **`FUNCAO_ID` é o setor. O cargo é outro eixo. A permissão é um terceiro.**

- **Setor:** `tboperador.FUNCAO_ID` → `tbfuncao` (1 Implantação, 2 Suporte, 3 Desenvolvimento,
  4 ADM/Financeiro, 5 CIAA). Eixo do Kanban, do dono do card e do escopo de leitura.
- **Cargo:** `tboperador.CARGO_ID` → `tbcargo`. As 9 funções da especificação + `VISUALIZADOR`.
- **Permissão:** `tbperm_cargo` (grupo) e `tbperm_operador` (exceção), por recurso, ação e escopo.

Ação: `Ler=1, Criar=2, Editar=4, Excluir=8, Aprovar=16, Gerir=32, Auditar=64`.
Escopo: `1=próprio · 2=função/setor · 3=tudo`.
Motor: GESTOR_MASTER → bypass; exceção individual (negativa vence); permissão do cargo;
nada → negado. Consumidores: policy ASP.NET (`[Authorize(Policy = "perm:...")]`),
`GET /api/v1/permissoes/minhas` e o guard do Angular. 403 auditado em ponto único
(`AuditoriaNegacaoHandler`).

## Estado real verificado em 2026-11-05

| Fase / item | Status | Evidência |
|---|---|---|
| 0 — decisões e mapa (C1–C6) | concluída | C1–C6 fechados; `tbfuncao` alinhada; `tbcargo` + `CARGO_ID` |
| 1 — modelo e carga | concluída | `Permissoes_*_Idempotente.sql` aplicados em HML; `AppDbContext` com `Cargos`, `PermissoesRecurso`, `PermissoesCargo`, `PermissoesOperador`; carga: 8 `GESTOR_MASTER`, 1 `TECH_LEAD_IMPLANTACAO` |
| 2 — motor e policies | concluída | `PermissaoService`, `PermissaoHandler`, `PermissaoPolicyProvider`, `AuditoriaNegacaoHandler`, `GET /permissoes/minhas`; 558 testes verdes na época |
| 3 — operadores/cargos (API + tela) | concluída | `/sistema/permissoes`; `GET/PUT /permissoes/matriz/{cargoId}`; sidebar filtra por `/permissoes/minhas`; `header-nav.config.ts` usa `recurso` |
| 4 — fechar buracos / escopo de leitura | **parcial** | `DatabaseController.cs:14` → `perm:aba.ferramentas.banco-dados:Ler` (fechado). `AcessosController` segue `[Authorize]` **deliberado** (prova de senha; sem papel). Restam **28 `[Authorize]` simples vs 22 policies** — triagem pendente. Escopo de leitura não auditado por completo |
| 5 — fuso único (`TimeZoneHelper`) | concluída | `TimeZoneHelper.cs` + `TimeZoneHelperTests`; uso em `AgendaService` (e demais serviços de data). Varredura residual de usos UTC diretos pendente |
| 6 — navegação v2 / Base de Conhecimento / "Agentes" | **superseded — concluída** | `/base-conhecimento` e visões por setor entregues (`features.routes.ts:102-104,189-195`, `permissaoGuard`); chat = **iframe + SSO** (`cb36ad3`, 2026-11-03). "Agentes" como nome de `/chat` não existe mais; a redação original está obsoleta |
| 7 — `api/v1/tarefas` + alias da rota antiga | **pendente** | 0 ocorrências de `v1/tarefas` no backend |
| 8 — visão geral por setor | concluída | `features.routes.ts`: `suporte/visao-geral` (`funcaoId=2`), idem desenvolvimento e ciaa; `/ciaa/projetos`; entregue em `6232c7d` |
| 9 — auditoria admin + `dotnet test` no gate | **parcial** | `validate.ps1` executa `dotnet test` (etapa 5). **Auditoria de alteração da matriz ausente**: `PermissoesController.SalvarMatriz` chama `SubstituirMatrizAsync` sem gravar trilha (quem/quando/diff) |
| 10 — regressão e aceite | **pendente** | depende das fases acima |

## O que falta (backlog ordenado)

1. **Triagem dos 28 `[Authorize]` simples**: para cada um, aplicar policy RBAC ou registrar
   justificativa explícita no código (ex.: endpoint público de login/refresh/me permanecem
   fora de policy por desenho).
2. **Escopo de leitura**: auditoria completa das consultas (tarefas, projetos, cards,
   apontamentos) contra o escopo `1/2/3`; corrigir as que ignoram o escopo.
3. **Fase 7**: criar rota `api/v1/tarefas` com alias da rota atual; cobrir com testes.
4. **Fase 9**: trilha de auditoria da alteração da matriz — evoluir `tbauditoriaimplantacao`
   (Q26/C6), sem tabela nova; registrar ator, cargo-alvo, antes/depois e IP.
5. **Fase 5 residual**: varrer usos diretos de `DateTime.UtcNow`/`Now` fora do helper.
6. **Fase 10**: regressão e aceite (QA + roteiro de smoke pós-deploy).

## Fora do escopo (mantido)

- Migrar `tboperador.SENHA` para KDF com sal (projeto próprio).
- Mock API / Mock Request.
- Unificar o "gestor geral" entre Ideias e Kanban (2 sentidos opostos; decisão de domínio).
- Trocar o módulo Acessos de prova de senha por papel (decisão deliberada).
- Rotacionar `Jwt:Key` (bloqueado pela credencial do IIS em `.254`).

## Registro de execução (condensado)

- **2026-10-21**: plano criado (Q1–Q30); baseline commitada; modelos + script idempotente
  criados; **incidente**: build publicado no `.130` antes do schema → login 500;
  `Permissoes_Idempotente.sql` aplicado em `dbBUSINESS_HML`, `tbfuncao` alargada/alinhada,
  login voltou a 401; Fases 2 e 3 implementadas (558 testes verdes); deploy HML + smoke verde.
- **2026-10-28**: revisão contra o código — Fases 0–4 implementadas; 2, 5 e 6 parciais
  (status registrado nos planos apagados).
- **2026-11-03**: `cb36ad3` — chat virou iframe + SSO (`ChatSsoController`); removidos
  `RagProxyController`, `ChatPerguntarService`, catálogo de agentes e o widget JOTA;
  `5988c65` limpou a documentação obsoleta.
- **2026-11-05**: planos `rbac/` e `reestruturacao-navegacao-permissoes/` unificados neste
  arquivo; reverificação: `DatabaseController` já tem policy; restam 28 `[Authorize]`
  simples; auditoria de alteração de matriz ausente; Fase 7 não iniciada.

## Riscos

| Risco | Mitigação |
|---|---|
| HML ≠ produção em `tbfuncao` | alinhar por script idempotente; validar antes do publish (já alinhado em 2026-10-21; revalidar a cada publish) |
| Carga de cargo errada | teste de paridade + mapa aprovado + backup |
| Migração sem staging (ADR 003) | backup + `RESTORE VERIFYONLY` + script idempotente antes do publish |
| Senha `varchar(10)` | aceito com aviso (Q13); dívida registrada |
| Permissão nova sem recurso no catálogo | `tbpermissao_recurso` semeada pelo extrator; rota nova só passa com recurso |
