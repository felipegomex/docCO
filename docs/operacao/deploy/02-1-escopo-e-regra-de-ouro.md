---
id: sec.02-1-escopo-e-regra-de-ouro
titulo: "1. Escopo e regra de ouro"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 304
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 1. Escopo e regra de ouro

O fluxo obrigatório é:

```text
subir interno
  → validate
  → commit/push pelo fluxo normal
  → validate
  → package
  → publish
  → smoke
```

Regra de ouro: **não publique sem `validate.ps1` verde no commit atual**.

O `validate.ps1` apenas avisa quando o working tree está sujo; ele não bloqueia. Portanto, a limpeza do working tree é uma obrigação do operador, não uma garantia do script.

Skills opcionais:

- `subir interno` para desenvolvimento;
- `validar` para o gate de build;
- `deploy limpo` para automatizar validate → package → publish → smoke.

Os scripts são a fonte final do comportamento. Consulte [`subir-interno`](../../../.opencode/skills/subir-interno/SKILL.md), [`validar`](../../../.opencode/skills/validar/SKILL.md) e [`deploy-limpo`](../../../.opencode/skills/deploy-limpo/SKILL.md).

