---
id: sec.03-1-login-base
titulo: "1. Login/Base"
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
tokens_estimados: 1780
ficha: docs/modulos/auth-rbac-sistema.md
---
> Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 1. Login/Base

### LB-01 — Entrar na Central

- **Objetivo:** Permitir que um operador autenticado entre e volte à rota solicitada.
- **Pré-condições:** operador ativo no backend; formulário disponível em `/login`; opcionalmente um `returnUrl` válido.
- **Passos:** 1. Informar `Usuário` e `Senha`. 2. Marcar ou desmarcar `Lembrar meu acesso`. 3. Clicar em `Entrar`.
- **Resultado esperado:** credenciais válidas redirecionam para o `returnUrl` quando diferente de `/`; sem link direto, redirecionam para `/`, para qualquer perfil. Credenciais inválidas exibem mensagem genérica e não criam sessão. O access token e o usuário permanecem em memória; o backend envia o cookie HttpOnly `cc_refresh`. Com `Lembrar meu acesso` marcado, o cookie recebe `MaxAge` de quatro horas; sem a opção, permanece cookie de sessão.
- **Estado atual/bloqueio:** limitação conhecida de configuração: `cc_refresh` está com `Secure=false`; o cenário de produção sob HTTPS exige revisão desse atributo.
- **Rotas/endpoints/tabelas:** `/login`; `POST /api/v1/auth/login`, público, com corpo `{ usuario, senha, lembrarAcesso }`; tabelas `tboperador` e `tbrefreshtoken`.
- **Status/automação:** **cenário documentado**; não há teste automatizado de login/refresh na árvore de trabalho.

### LB-02 — Renovar, expirar e encerrar a sessão

- **Objetivo:** Manter a sessão após recarregar a página e encerrá-la com segurança quando a renovação falhar.
- **Pré-condições:** cookie `cc_refresh` válido ou expirado; acesso a uma rota protegida; sessão iniciada por um operador.
- **Passos:** 1. Recarregar uma rota protegida. 2. Observar a tentativa de renovação e a restauração do usuário. 3. Simular/observar uma resposta 401 ou usar `Sair`. 4. Tentar abrir novamente uma rota protegida.
- **Resultado esperado:** após F5, o token em memória pode ser repovoado pelo refresh e o Header volta a mostrar o usuário. O interceptor envia cookies e Bearer. Se o backend rejeitar o refresh, ele limpa `cc_refresh` e `cc_lembrar`, enquanto o frontend limpa o estado em memória; a requisição original falha e a próxima navegação protegida redireciona para `/login`. Com Bearer válido, o logout autenticado deve revogar o refresh antes de limpar os cookies. A tela de login exibe a mensagem de sessão expirada quando a sinalização foi criada pelo frontend.
- **Estado atual/bloqueio:** limitação conhecida: o `AuthService` chama `/auth/logout`, mas o interceptor exclui `/auth/` do envio de Bearer; quando o token já não está disponível, o endpoint `[Authorize]` pode responder 401 e a revogação do refresh não fica garantida. Falha de rede também pode impedir a revogação.
- **Rotas/endpoints/tabelas:** `POST /api/v1/auth/refresh` com cookie `cc_refresh`; `POST /api/v1/auth/logout` com `[Authorize]`; `tbrefreshtoken` e `tboperador`. O `GET /api/v1/auth/me` existe no backend, mas não é consumido pelo fluxo atual do frontend.
- **Status/automação:** **cenário documentado**; não há teste automatizado específico de sessão.

### LB-03 — Usar a Home operacional

- **Objetivo:** Oferecer uma entrada rápida para busca, ações operacionais e próximos compromissos.
- **Pré-condições:** operador autenticado; backend da Agenda disponível ou em estado de falha controlado.
- **Passos:** 1. Abrir `/`. 2. Ler a saudação e usar `Buscar na Central...`. 3. Testar os seis acessos rápidos. 4. Ler a agenda dos próximos sete dias e abrir `Ver agenda`.
- **Resultado esperado:** a Home mostra saudação com o primeiro nome, seis atalhos numa linha só (grade de 6 colunas; 3 colunas abaixo de 1400px e 1 coluna abaixo de 900px) e até cinco eventos da janela de sete dias. Falha da Agenda resulta em lista vazia, sem dados fictícios. Os blocos de “Continue de onde parou” e “Mais utilizados” não são renderizados no código atual.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a listagem da Agenda depende de o endpoint responder ou tratar a falha conforme o estado vazio.
- **Rotas/endpoints/tabelas:** `/`; `GET /api/v1/agenda/eventos?inicio=X&fim=Y`, autenticado; leitura de `tbagenda` e das tabelas de apoio de operador/tipo. A busca da Home é local e não tem endpoint.
- **Status/automação:** **cenário documentado**; não há teste automatizado da Home.

### LB-04 — Buscar no Header e na Home

- **Objetivo:** Encontrar páginas e conteúdo estático com uma busca compartilhada e acessível por teclado.
- **Pré-condições:** operador autenticado; Header ou campo da Home visível.
- **Passos:** 1. Digitar um termo. 2. Abrir um resultado com clique ou setas + `Enter`. 3. Usar `Escape`, `Tab` e `Ctrl+K`/`Cmd+K`. 4. Digitar um termo sem resultado.
- **Resultado esperado:** a consulta é compartilhada entre as duas origens, com no máximo oito resultados; `ArrowUp`/`ArrowDown` percorrem as opções, `Enter` navega, `Escape` fecha sem retirar o foco e o atalho abre, foca e seleciona o campo do Header. A semântica `combobox`/`listbox`/`option` e o status acessível permanecem expostos.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a validação existente é unitária e não substitui um teste de navegador.
- **Rotas/endpoints/tabelas:** Header e Home; nenhum endpoint HTTP e nenhuma tabela, pois `BuscaIndexService` monta o índice no navegador a partir dos acervos estáticos.
- **Status/automação:** **validado em teste** por `global-search.component.spec.ts` e `busca.service.spec.ts`; a cobertura é unitária e não representa execução desta revisão nem teste end-to-end.

### LB-05 — Navegar no layout autenticado

- **Objetivo:** Usar Header, Sidebar, breadcrumb, tema, rodapé e layouts sem perder o contexto da rota.
- **Pré-condições:** sessão válida; uma tela autenticada carregada.
- **Passos:** 1. Recolher/expandir a Sidebar. 2. Navegar por um link do menu e observar breadcrumb e outlet. 3. Alternar o tema. 4. Abrir o menu do usuário e sair.
- **Resultado esperado:** `MainLayout` monta Header, Sidebar, breadcrumb, Footer, outlet e o FAB do JOTA; `AdminLayout` é o shell separado das rotas administrativas. A escala é 80% fora de `/login`. O logout chama o endpoint de autenticação e redireciona para `/login`.
- **Estado atual/bloqueio:** limitação conhecida: a chamada de logout usa o mesmo interceptor que não adiciona Bearer a `/auth/`; a navegação local ocorre, mas a revogação no servidor depende de o backend receber uma requisição autorizada.
- **Rotas/endpoints/tabelas:** todas as rotas sob `authGuard`; `POST /api/v1/auth/logout` com `[Authorize]`; `tbrefreshtoken` para revogação. Não há endpoint direto de layout.
- **Status/automação:** **cenário documentado**; `app.component.spec.ts` cobre apenas criação da aplicação e presença do `router-outlet`, não o fluxo completo de navegação.

