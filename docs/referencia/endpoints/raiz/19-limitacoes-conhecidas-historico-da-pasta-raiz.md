---
id: raiz.19
titulo: "Limitações conhecidas (histórico da pasta raiz)"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 132
ficha: docs/modulos/README.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

## Limitações conhecidas (histórico da pasta raiz)

| Divergência | Estado vigente |
|---|---|
| `validacao` 5/min + `ClienteObrigatorio` | código atual: 50/100 por minuto, sem esse sinalizador no cadastro |
