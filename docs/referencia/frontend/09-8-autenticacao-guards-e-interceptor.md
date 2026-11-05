---
id: sec.09-8-autenticacao-guards-e-interceptor
titulo: "8. Autenticação, guards e interceptor"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 1284
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 8. Autenticação, guards e interceptor

### 8.1 Armazenamento

| Item | Comportamento |
|---|---|
| Access token | Somente em `TokenStorageService.accessToken`, em memória |
| Usuário | Em memória no storage e propagado por `BehaviorSubject` em `AuthService` |
| Refresh token | Não é salvo pelo Angular; o backend o envia em cookie HttpOnly `cc_refresh` |
| Recuperação após F5 | O guard aguarda `authInitialized$` e o Angular chama `/auth/refresh` com credenciais |
| Persistência visual da expiração | `sessionStorage.sessaoExpirada`, somente uma flag de mensagem |

### 8.2 Fluxo de login

1. `LoginComponent` valida campos obrigatórios com Reactive Forms.
2. `AuthService.login()` envia `usuario`, `senha` e `lembrarAcesso` para `/auth/login`.
3. A resposta contém access token e usuário; o frontend ignora qualquer conteúdo de refresh token porque ele é gerenciado por cookie.
4. `persistSession()` guarda access token e usuário em memória e inicia a verificação periódica.
5. O login respeita `returnUrl`; sem deep link, navega para `/`.

### 8.3 Refresh e interceptor

[`auth.interceptor.ts`](../../../frontend/src/app/core/interceptors/auth.interceptor.ts):

1. aplica `withCredentials: true` a toda requisição;
2. não adiciona Bearer quando a URL contém `/auth/`;
3. nas demais, adiciona `Authorization: Bearer <access token>` quando disponível;
4. em resposta 401, chama `AuthService.refreshTokenSingleFlight()`;
5. com novo token, repete a requisição original uma vez; sem token, propaga o erro.

O refresh usa uma Promise compartilhada para impedir múltiplas renovações simultâneas. A tela de login não chama `GET /auth/me`; depois do refresh, o usuário vem no próprio corpo da resposta ou é reconstruído a partir das claims do JWT.

### 8.4 Guards

| Guard | Browser | Observação |
|---|---|---|
| `authGuard` | Aguarda auth inicializada, aceita access válido ou tenta refresh; em falha manda para `/login` com `returnUrl` | Retorna `true`; a proteção real ocorre no cliente; sem SSR, executa somente no browser |
| `adminGuard` | Exige perfil `Administrador`; perfil não administrador vai para `/` | Retorna `true`; sem SSR, executa somente no browser |

A proteção de API continua sendo aplicada pelo backend JWT. No cliente, `isAuthenticated()` verifica somente presença do token e `exp`; assinatura, issuer e audience não são validados no navegador. `AuthService.hasRole()` compara **o rótulo** e **a letra de `PERFIL_ID`** (`Usuario.perfilId`, preenchido pela claim `perfil_id`, que o backend já emite **normalizada** — trim + maiúsculas), de modo que o item de menu escrito como `roles: ['F']` continua funcionando depois do rename do rótulo para `Financeiro`; `adminGuard` é estrito.

### 8.4.1 `Usuario` e navegação por papel

[`auth.model.ts`](../../../frontend/src/app/core/models/auth.model.ts) define `PerfilUsuario = 'Usuario' | 'Editor' | 'Administrador' | 'Suporte' | 'Financeiro' | 'Produtor'` — rótulos devolvidos por `AuthService.MapearPerfil` no backend. `Usuario` carrega `funcaoId`, `funcaoDescricao`, `funcaoClassificacao`, `perfilId`, `ehAdmin` e os três campos do **cargo RBAC** (`cargoId`, `cargoCodigo`, `cargoDescricao`, alimentados pelas claims `cargo_id`, `cargo_codigo` e `cargo_descricao`). O cargo é um eixo **separado** do setor: `funcaoId` continua sendo `FUNCAO_ID`; a permissão efetiva vem de `GET /permissoes/minhas`. O frontend **não** lê mais `setor_id`/`setor_nome`: as claims `setor_id`/`setor_nome` foram removidas do JWT, e `TokenPayload` declara `funcao_id`, `funcao_descricao` e `funcao_classificacao`.

[`header-nav.config.ts`](../../../frontend/src/app/shared/config/header-nav.config.ts) declara `export type NavRole = PerfilUsuario | 'A' | 'S' | 'F' | 'P'`, porque `roles` continua aceito como compatibilidade. Desde 2026-10-22 o filtro principal da sidebar é o **recurso RBAC**: cada item declara `recurso: 'aba.*'` e o menu aplica `PermissoesService.pode(recurso, Ler)`. Cada setor tem o seu item **Kanban** com `?funcaoId=`; o item "Kanban ADM" saiu do menu em 2026-10-20 e o recorte `?perfil=` continua existindo apenas como deep link. O inventário completo das **8 seções, 42 itens-folha e 32 URLs distintas** está em §4.10.

### 8.5 Logout e sessão expirada

`logout()` envia `POST /auth/logout` sem aguardar a resposta, limpa token/usuário em memória e navega para `/login`. Como o interceptor exclui todas as URLs `/auth/` do Bearer, essa chamada é feita apenas com credenciais/cookie, embora o endpoint exija JWT; o estado local é limpo independentemente da resposta. Define `sessionStorage.sessaoExpirada` quando há redirecionamento. A verificação periódica roda a cada cinco minutos; se o access token estiver expirado, encerra a sessão, e se ainda estiver válido, tenta renovar.

