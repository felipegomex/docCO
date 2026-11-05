---
id: sec.01-criterios-transversais-confirmados
titulo: "Critérios transversais confirmados"
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
tokens_estimados: 449
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# Critérios transversais confirmados

- Telas autenticadas usam `authGuard`; `/admin` usa também `adminGuard` e exige perfil `Administrador`. As telas de `/sistema` usam `permissaoGuard` por recurso (`aba.sistema.funcoes-administrativas` e `aba.sistema.operadores`), alimentado por `GET /api/v1/permissoes/minhas`; o backend revalida com policies `perm:<recurso>:<acao>`.
- O access token e o usuário ficam somente em memória. O refresh token fica no cookie HttpOnly `cc_refresh`, com `SameSite=Strict` e `Secure=false` na configuração atual; o interceptor envia todas as requisições com credenciais (`withCredentials`), inclusive as chamadas de autenticação.
- A escala global `html.scaled` é 80% (`1rem = 12,8px`) em todas as telas, exceto `/login`, que permanece em 100%.
- A listagem de Projetos usa o layout compacto de três colunas: filtros, conteúdo e resumo; a lista de projetos é vertical e cada card mostra as nove etapas fixas.
- Conteúdo estático, buscas locais, `localStorage` e `sessionStorage` não são tratados como endpoint ou persistência de backend.
- Tarefa com projeto exige etapa do projeto **do mesmo projeto**; o backend recusa etapa sem projeto na criação e zera a etapa da tarefa sem projeto na atualização. O formulário de Tarefa mantém a etapa sempre visível, desabilitada sem projeto, durante o carregamento e em caso de falha de carga, e o `PUT` de tarefa não altera o projeto.

