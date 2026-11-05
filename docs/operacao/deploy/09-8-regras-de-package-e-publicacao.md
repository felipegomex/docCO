---
id: sec.09-8-regras-de-package-e-publicacao
titulo: "8. Regras de package e publicação"
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
tokens_estimados: 151
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 8. Regras de package e publicação

- `robocopy /E` no package não espelha nem limpa; `/MIR` só é usado no IIS.
- Códigos 8 ou superior falham; ainda confirme artefatos, `BUILD_INFO` e a ausência de `app_offline.htm` residual.

