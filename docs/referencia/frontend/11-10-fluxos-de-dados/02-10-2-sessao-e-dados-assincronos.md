---
id: 11-10-fluxos-de-dados.02
titulo: "10.2 Sessão e dados assíncronos"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, fluxos]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 213
ficha: docs/modulos/README.md
---
> Parte de [10. Fluxos de dados](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 10.2 Sessão e dados assíncronos

- Componentes de formulário e páginas chamam serviços injetados.
- Serviços HTTP devolvem `Observable`; serviços locais devolvem arrays/síncrono ou observables conforme o caso.
- Páginas mantêm estado em signals, properties ou `BehaviorSubject` conforme o módulo.
- Falhas de APIs são tratadas localmente nos componentes; o interceptor apenas executa refresh/replay e não regra de negócio.
- Busca global e Home compartilham `BuscaService`; a fonte do índice é o bundle estático do frontend.

