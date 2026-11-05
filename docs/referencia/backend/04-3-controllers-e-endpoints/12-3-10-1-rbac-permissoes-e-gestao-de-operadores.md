---
id: 04-3-controllers-e-endpoints.12
titulo: "3.10.1 RBAC, permissões e gestão de operadores"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1584
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../../modulos/auth-rbac-sistema.md).

### 3.10.1 RBAC, permissões e gestão de operadores

Três controllers, todos em `api/v{version:apiVersion}` com `[Authorize]` na classe. O detalhe das rotas está em [`endpoints/raiz.md`](../../endpoints/raiz/INDEX.md).

| Controller | Rota base | Operações | Autorização |
|---|---|---:|---|
| `PermissoesController` | `/api/v1/permissoes` | 5 | `[Authorize]`; `GET /minhas` devolve o conjunto efetivo do operador do token; `GET /recursos` chama `EhMasterAsync` e responde **403** para quem não é `GESTOR_MASTER`; `GET /cargos-gerenciaveis` e `GET`/`PUT /matriz/{cargoId}` usam `PodeGerenciarCargoAsync` (Master ou Tech Lead do setor/nível) |
| `OperadoresController` | `/api/v1/operadores` | 6 | `GET` (lista e detalhe) só exige JWT; `POST` usa `perm:aba.sistema.operadores:Criar`, `PUT`, `PATCH /status` e `PATCH /senha` usam `perm:aba.sistema.operadores:Editar` |
| `CargosController` | `/api/v1/cargos` | 1 | `[Authorize]`; lista os cargos **ativos** ordenados por `CARGO_ID` |

**Policies `perm:<recurso>:<acao>`.** [`PermissaoPolicyProvider`](../../../../backend/Central_BackEnd/Authorization/PermissaoPolicyProvider.cs) reconhece o prefixo `perm:` (sem diferenciar caixa), faz `Split(':', 3)`, converte a ação pelo nome do enum `AcaoPermissao` (`Ler`, `Criar`, `Editar`, `Excluir`, `Aprovar`, `Gerir`, `Auditar`) e monta uma policy com `RequireAuthenticatedUser()` + `PermissaoRequirement(recurso, acao)`. Policies que não começam com `perm:` seguem no `DefaultAuthorizationPolicyProvider`. O nome do recurso é a chave de `tbpermissao_recurso` (ex.: `aba.sistema.operadores`).

[`PermissaoHandler`](../../../../backend/Central_BackEnd/Authorization/PermissaoHandler.cs) resolve o operador por `ClaimTypes.NameIdentifier` (fallback `sub`) e chama `IPermissaoService.ResolverAsync(operadorId, recurso, acao, funcaoAlvo: null)`. Não há decisão de linha no handler: dono, gestor e função do registro continuam nos serviços de domínio.

**Hierarquia da matriz (2026-10-27).** `Cargo` ganhou `Nivel` (`CARGO_Nivel`) e `FuncaoId` (`CARGO_FuncaoId`): Master 4 (global), Tech Lead 3 (no setor do cargo), comum 2 e Visualizador 1. `Nivel` é **`byte`** no CLR porque a coluna é `tinyint`: com `int`, o EF lê por `SqlDataReader.GetInt32` e derruba o login (`InvalidCastException` `Byte -> Int32`) ao materializar o `Cargo` — os consumidores continuam usando `int` por conversão implícita. `PodeGerenciarCargoAsync(operadorId, cargoId)` carrega o cargo do operador e o alvo, e `PodeGerenciar` aplica a regra única:

- `GESTOR_MASTER` gerencia **qualquer cargo, exceto `GESTOR_MASTER`** — o Master não entra na matriz porque já tem bypass;
- Tech Lead (`CARGO_Nivel > 0`) gerencia somente cargo com **`CARGO_FuncaoId` igual ao do seu próprio cargo** e **nível inferior** ao dele;
- qualquer outro cargo é negado.

`ListarCargosGerenciáveisAsync` lista os cargos **ativos** que passam nessa regra (ordem por `CARGO_ID`). `SubstituirMatrizAsync` desativa apenas as linhas com `PEC_FuncaoId IS NULL`, preservando as concessões por setor (ex.: CIAA → Ideias).

**Auditoria do 403.** [`AuditoriaNegacaoHandler`](../../../../backend/Central_BackEnd/Authorization/AuditoriaNegacaoHandler.cs) é o `IAuthorizationMiddlewareResultHandler` registrado. Quando o resultado é `Forbidden` e a policy tem `PermissaoRequirement`, ele enfileira em `IAuditoriaFila` um `EventoAuditoria` com `Entidade = "Autorizacao"`, `Acao = "ACESSO_NEGADO"` e `Observacao = "recurso=…;acao=…;ip=…;cargo=…;funcao=…"`, lendo `cargo_codigo` e `funcao_id` das claims. É o **único** ponto de registro de acesso negado: controllers e serviços não repetem `Forbid()` para auditar.

**Gestão de operadores.** [`OperadorAdminService`](../../../../backend/Central_BackEnd/Services/Shared/OperadorAdminService.cs) opera sobre `tboperador` e não tem `DELETE`:

| Regra | Implementação confirmada |
|---|---|
| Quem administra | `GESTOR_MASTER` administra qualquer operador/cargo/setor. `TECH_LEAD_*` (três códigos) administra só operadores do **próprio setor** (`FuncaoId`) e cargos de nível **abaixo** do Tech Lead. Qualquer outro cargo recebe `403` (`"Somente Master e Tech Lead administram operadores."`). |
| Nível hierárquico | `Nivel(codigo)`: `GESTOR_MASTER` = 100; `TECH_LEAD_IMPLANTACAO`/`TECH_LEAD_DESENVOLVIMENTO`/`TECH_LEAD_CIAA` = 80; `SUPORTE_N2` = 60; `SUPORTE_N1`/`IMPLANTACAO`/`DESENVOLVEDOR`/`CIAA` = 40; `VISUALIZADOR` = 10; desconhecido = 0. |
| Master alterando Master | Só o Master altera Master ou outro Tech Lead; o Tech Lead recebe `403` (`"Tech Lead nao altera Master nem outro Tech Lead."`). |
| Autoproteção | Ninguém altera o **próprio cargo** (`403`) nem inativa a si mesmo (`403`). |
| Exclusão | **Não existe** endpoint `DELETE`: a saída é inativar (`SE_ATIVO = 'N'`) via `PATCH /{operadorId}/status`; a lista mantém o registro. |
| Senha | Obrigatória e com **no máximo 10 caracteres** (`TBOPERADOR.SENHA varchar(10)`); acima disso, `400`. O valor continua sendo gravado no formato atual (comparado em texto pelo login) e **não** aparece na auditoria — o evento `SENHA` registra só `SenhaAlterada = true`. |
| E-mail | Opcional, `<= 100` caracteres, precisa conter `@` e não pode começar/terminar com ele; a unicidade é conferida **sem diferenciar caixa** e devolve `409` (`EMAIL_DUPLICADO`). |
| `OPERADOR_ID` | Obrigatório, `<= 15` caracteres; duplicado devolve `409` (`OPERADOR_DUPLICADO`). |
| Cargo e setor | `CargoId` é obrigatório na criação/edição e precisa existir **ativo**; `FuncaoId` é opcional e, quando informado, precisa existir ativo. |
| Ponte com o Kanban legado | Criar ou **promover** a `GESTOR_MASTER` grava `SE_ADMIN = true` e `PERFIL_ID = 'A'` (o `FuncaoAuthorizationService` ainda usa essas duas portas como **ponte** para o Master). **Rebaixar não limpa** `SE_ADMIN`/`PERFIL_ID` automaticamente, para não revogar um admin legado sem revisão. |
| Auditoria | `CREATE`, `UPDATE`, `ATIVAR`, `INATIVAR` e `SENHA` vão para `tbauditoriaimplantacao` com o retrato do operador (sem senha) e a observação `OPERADOR_ID=…`. |

