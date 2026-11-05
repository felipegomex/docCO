---
id: 11-10-fluxos-de-dados.03
titulo: "10.3 Busca e acervos locais"
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
tokens_estimados: 165
ficha: docs/modulos/README.md
---
> Parte de [10. Fluxos de dados](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 10.3 Busca e acervos locais

`BuscaIndexService` agrega páginas, ferramentas, cursos, trilhas, consultas SQL, procedimentos, utilitárias, políticas e capítulos de onboarding. A filtragem remove acentos, exige todos os termos com mais de um caractere e navega para a rota do resultado. Não há chamada de busca ao backend nesse fluxo.

