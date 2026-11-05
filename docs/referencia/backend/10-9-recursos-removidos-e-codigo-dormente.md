---
id: sec.10-9-recursos-removidos-e-codigo-dormente
titulo: "9. Recursos removidos e código dormente"
tipo: referencia
area: backend
sistema: central
tags: [backend, aspnet, controllers, services]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 421
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/backend.md](../backend.md).

# 9. Recursos removidos e código dormente

### 9.1 Etapas Globais

Não há modelo `Etapa`, `IETapaService`, controller `/api/v1/implantacao/etapas` nem tabela `IMPL_Etapa` no snapshot atual. A API vigente é subconjunto de `/api/v1/implantacao/projetos/.../etapas`. A migração `RemoveEtapaAntiga` formaliza a remoção.

### 9.2 Itens existentes sem consumidor de endpoint

| Item | Situação |
|---|---|
| `IAuditoriaImplantacaoService.ObterHistoricoAsync` | Implementado, mas o histórico de Tarefas consulta `tbauditoriaimplantacao` diretamente |
| filtro `equipe` em Tarefas | Parâmetro permanece no DTO/controller, mas não é aplicado em `TarefaService` |
| `/api/rag-proxy`, `/api/chat/agentes`, `/api/chat/perguntar` | Removidos no `cb36ad3` (2026-11-03); o contrato de chat vigente é `GET /api/chat/sso` (iframe para o AnythingLLM). |
| `Models/Acessos/PasswordValidationRequest.cs` e `PasswordValidationResponse.cs` | **Código morto**: o gate é `POST /acessos/clientes/{id}/autorizar`, com `AutorizarRequest`/`AutorizarResponse`. Nenhum endpoint consome esses dois modelos |
| `AdminDashboardService` / `AdminDashboardController` | O endpoint `GET /api/v1/admin/dashboard` existe e está registrado no DI, mas **nenhuma tela do frontend o consome**: o componente e o serviço do frontend foram removidos em 2026-10-20 (Fase 0) |

