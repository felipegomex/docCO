---
id: sec.13-12-branches-tags-e-versionamento
titulo: "12. Branches, tags e versionamento"
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
tokens_estimados: 313
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 12. Branches, tags e versionamento

| Branch | Uso |
|---|---|
| `developer` | desenvolvimento e push direto pelo fluxo normal |
| `master` | produção e releases, alimentado por merge de `developer` |

Branches manuais `backup-*` existem apenas para recuperação histórica e não fazem parte do fluxo diário. Tags `vX.Y.Z` substituem branches de backup.

### Regras

1. Desenvolva e valide em `developer`.
2. Faça push do commit validado.
3. Revalide o commit.
4. Publique conforme o pipeline obrigatório.
5. Promova `developer` para `master` por merge revisado.
6. Só crie tag depois de confirmar que a versão está mergeada em `master`.
7. Crie tag anotada `vX.Y.Z`, com descrição, e publique no remoto.
8. Não mova, reutilize ou exclua tag de versão.
9. A versão SemVer (`vX.Y.Z`) acompanha o `Version` do `BUILD_INFO`, o `versao` do `version.json`, o `APP_VERSION` e o `APP_CONFIG.versao`; a exibição no Guia do Sistema usa o valor publicado no build.

Para consulta pontual, use `git branch --all` e `git tag --list`; este runbook não fixa uma lista de tags que envelhece.

