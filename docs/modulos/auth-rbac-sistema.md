---
id: mod.auth-rbac-sistema
titulo: "Auth, RBAC e Sistema"
tipo: modulo
area: seguranca
sistema: central
tags: [auth, login, jwt, refresh, rbac, permissao, cargo, operador, auditoria, sistema]
dono: sistema
status: ativo
revisado_em: "2026-10-29"
fonte: [backend/Central_BackEnd/Controllers/AuthController.cs, backend/Central_BackEnd/Authorization/PermissaoPolicyProvider.cs, backend/Central_BackEnd/Services/Permissoes/PermissaoService.cs, frontend/src/app/core/services/auth.service.ts, frontend/src/app/core/guards/permissao.guard.ts, frontend/src/app/features/sistema, frontend/src/app/features/faq-sistema/faq-sistema.component.ts]
testes: [backend/Central_BackEnd.Tests/Api/AuthContratosTests.cs, backend/Central_BackEnd.Tests/PermissoesServiceTests.cs, backend/Central_BackEnd.Tests/OperadorAdminServiceTests.cs]
tokens_estimados: 1800
---

# Auth, RBAC e Sistema

## 1. Quando usar este card

Login/logout/refresh, cookie de sessão, JWT, 401/403, policy `perm:`, matriz de
permissão por cargo, CRUD de operador, cargos, auditoria, `/sistema` e o Guia do
Sistema (`/faq-sistema`).

## 2. Propósito e escopo

Autentica o operador (JWT + refresh em cookie), resolve a permissão por
cargo/recurso e mantém operadores e a matriz administrativa. Não cobre a
permissão **por função** do Kanban (`FuncaoAuthorizationService`, `kanban.md`)
nem a conta do Acessos Empresas (`acessos.md`).

## 3. Rotas do frontend

| URL | Componente | Fonte |
|---|---|---|
| `/login` | `LoginComponent` | `frontend/src/app/app.routes.ts` |
| `/sistema/funcoes-administrativas` | `FuncoesAdministrativasComponent` | `frontend/src/app/features/sistema/sistema.routes.ts` |
| `/sistema/operadores` | `OperadoresComponent` | idem |
| `/sistema/permissoes` | `PermissoesComponent` | idem |
| `/auditoria` | `AuditoriaComponent` | `frontend/src/app/features/auditoria/auditoria.routes.ts` |
| `/faq-sistema` | `FaqSistemaComponent` | `frontend/src/app/features/faq-sistema/faq-sistema.component.ts` (declarada em `frontend/src/app/features.routes.ts`) |

## 4. Endpoints do backend

| Rotas | Autorização | Fonte |
|---|---|---|
| `POST /auth/login` (anônimo), `POST /auth/refresh` (cookie), `POST /auth/logout`, `GET /auth/me` | login/refresh anônimos; logout/me `[Authorize]` | `backend/Central_BackEnd/Controllers/AuthController.cs` |
| CRUD `/operadores` | `perm:aba.sistema.operadores:{Ler,Criar,Editar}` | `backend/Central_BackEnd/Controllers/OperadoresController.cs` |
| `/permissoes/minhas`, `/recursos`, `/cargos-gerenciaveis` | `[Authorize]` | `backend/Central_BackEnd/Controllers/PermissoesController.cs` |
| `GET/PUT /permissoes/matriz/{cargoId}` | `[Authorize]` + gerência | idem |
| `GET /cargos` | `[Authorize]` | `backend/Central_BackEnd/Controllers/CargosController.cs` |
| `GET /auditoria/acessos`, `/auditoria/implantacao` | `[Authorize]` | `backend/Central_BackEnd/Controllers/AuditoriaController.cs` |
| CRUD `/gestores` | `[Authorize]` | `backend/Central_BackEnd/Controllers/GestoresController.cs` |

## 5. Tabelas e entidades

| Classe → tabela | Papel | Fonte |
|---|---|---|
| `Operador` → `tboperador` | Usuário e cargo | `backend/Central_BackEnd/Models/Operador.cs` |
| `RefreshToken` → `tbrefreshtoken` | Refresh persistido | `backend/Central_BackEnd/Models/RefreshToken.cs` |
| `Cargo` → `tbcargo` | Cargo | `backend/Central_BackEnd/Models/Permissoes/Cargo.cs` |
| `PermissaoRecurso` → `tbpermissao_recurso` | Catálogo de abas | `backend/Central_BackEnd/Models/Permissoes/PermissaoRecurso.cs` |
| `PermissaoCargo` → `tbperm_cargo`; `PermissaoOperador` → `tbperm_operador` | Herança e exceção | `backend/Central_BackEnd/Models/Permissoes/PermissaoCargo.cs` |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `AuthService` | Login, refresh, revogação e usuário | `backend/Central_BackEnd/Services/AuthService.cs` |
| backend | `PermissaoPolicyProvider` | Cria `perm:<recurso>:<ação>` | `backend/Central_BackEnd/Authorization/PermissaoPolicyProvider.cs` |
| backend | `PermissaoService` | Decisão, matriz e cargos | `backend/Central_BackEnd/Services/Permissoes/PermissaoService.cs` |
| backend | `OperadorAdminService` | CRUD de operador e senha | `backend/Central_BackEnd/Services/Shared/OperadorAdminService.cs` |
| backend | `JwtKeyProvider` | Fonte única da chave | `backend/Central_BackEnd/Services/JwtKeyProvider.cs` |
| backend | `BruteForceGuard` | Bloqueio de tentativas | `backend/Central_BackEnd/Services/BruteForceGuard.cs` |
| frontend | `AuthService` | Sessão, refresh single-flight e logout | `frontend/src/app/core/services/auth.service.ts` |
| frontend | `PermissoesService` | Permissões efetivas | `frontend/src/app/core/services/permissoes.service.ts` |
| frontend | `permissaoGuard` | Proteção de rota | `frontend/src/app/core/guards/permissao.guard.ts` |

## 7. Regras de negócio

- Access é JWT; refresh vai no cookie `cc_refresh` (`SameSite=Strict`) e
  `cc_lembrar` decide a persistência (`AuthController#GravarCookieRefresh`).
- `JWT_KEY` e `Jwt:Key` podem divergir; `JwtKeyProvider` acusa e `JWT_KEY` vence.
- Permissão efetiva é cargo + exceção individual
  (`PermissaoService.ResolverAsync`); a policy é gerada por
  `PermissaoPolicyProvider`.
- Matriz só é editável para cargo gerenciável (`PodeGerenciarCargoAsync`).
- O recurso do operador é `aba.sistema.operadores`.

## 8. Autorização

- Rotas administrativas usam `perm:<recurso>:<ação>`, com recurso `aba.*`.
- `/sistema/*` espelha a mesma aba no frontend via `permissaoGuard`.
- `/auditoria` não tem guard de papel no cliente de propósito: o backend responde
  403 e a tela traduz.
- Login e refresh são anônimos por contrato; o refresh depende do cookie.
- `GestoresController` exige admin em código (`EhAdministrador` + `Forbid`).

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “login / mensagem 401” | `backend/Central_BackEnd/Controllers/AuthController.cs#Login` | `frontend/src/app/features/auth/pages/login/login.component.ts#LoginComponent` | `backend/Central_BackEnd.Tests/Api/AuthContratosTests.cs` |
| “refresh/cookie” | `backend/Central_BackEnd/Controllers/AuthController.cs#Refresh` | `frontend/src/app/core/services/auth.service.ts#AuthService` | `backend/Central_BackEnd.Tests/Api/AuthContratosTests.cs` |
| “nova aba/permissão” | `backend/Central_BackEnd/Authorization/PermissaoPolicyProvider.cs#PermissaoPolicyProvider` | `frontend/src/app/core/guards/permissao.guard.ts#permissaoGuard` | `backend/Central_BackEnd.Tests/PermissoesServiceTests.cs` |
| “matriz por cargo” | `backend/Central_BackEnd/Services/Permissoes/PermissaoService.cs#SubstituirMatrizAsync` | `frontend/src/app/features/sistema/pages/permissoes/permissoes.component.ts#PermissoesComponent` | `backend/Central_BackEnd.Tests/PermissoesContratosTests.cs` |
| “CRUD de operador” | `backend/Central_BackEnd/Services/Shared/OperadorAdminService.cs#CriarAsync` | `frontend/src/app/features/sistema/pages/operadores/operadores.component.ts#OperadoresComponent` | `backend/Central_BackEnd.Tests/OperadorAdminServiceTests.cs` |
| “Guia do Sistema / badge de versão” | - | `frontend/src/app/features/faq-sistema/faq-sistema.component.ts#FaqSistemaComponent`, `frontend/src/app/core/services/build-info.service.ts#BuildInfoService` | `frontend/src/app/features/faq-sistema/faq-sistema.component.spec.ts` |

## 10. Contratos que não podem quebrar

- Policy é `perm:<recurso>:<ação>`; mudar o formato derruba as rotas.
- Refresh **nunca** volta no corpo; só no cookie.
- Cookie é `SameSite=Strict` (proteção CSRF).
- `JWT_KEY` no ambiente da máquina é o contrato de produção.
- Senha do operador não é a senha do gate de Acessos.

## 11. Armadilhas conhecidas

- `PERFIL_ID` com minúsculas/espaços: `PerfilOperador.Normalizar`.
- Reciclar o pool do IIS apaga `JWT_KEY` do pool e derruba sessões.
- Aba renomeada no front e no back ao mesmo tempo; senão a permissão não casa.

## 12. Testes

- `AuthContratosTests`, `AuthJwtTests`, `PermissoesServiceTests`,
  `PermissoesCargoHierarquiaTests`, `PermissoesContratosTests`,
  `OperadorAdminServiceTests` e `PerfilOperadorTests`.
- QA: `docs/referencia/qa/03-1-login-base.md`,
  `docs/referencia/qa/08-3-3-sistema-rbac-e-operadores.md` e
  `docs/referencia/qa/04-2-conhecimento-ferramentas/02-cf-08-gerar-modelo-de-chamado.md` (CF-13 — Guia do Sistema).

## 13. Orçamento de leitura

Abra esta ficha; depois só o arquivo da seção 9 e o teste. Limite: 1 ficha,
4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- Exceção individual coberta por `PermissoesServiceTests` e contrato HTTP.
