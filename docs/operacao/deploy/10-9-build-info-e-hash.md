---
id: sec.10-9-build-info-e-hash
titulo: "9. `BUILD_INFO` e hash"
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
tokens_estimados: 413
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 9. `BUILD_INFO` e hash

`deploy/BUILD_INFO.txt` contém:

- `GitCommit`
- `GitBranch`
- `Ambiente` (`HML`, `PROD` ou `CUSTOM` quando `-Ambiente` não foi informado)
- `Servidor`
- `DestinoBackend` / `DestinoFront`
- `DataUtc`
- `Usuario`
- `BuildFrontend`
- `BuildBackend`
- `Maquina`
- `Version` (`1.0.0`, SemVer do frontend; gravado junto com o `version.json` do build)

O arquivo é regravado quando package executa ao menos um build. A publicação sem build preserva o arquivo existente e compara `GitCommit` com o HEAD atual. Cada package e cada publish também grava um recibo em `deploy/history/<AMBIENTE>-package-<timestamp>.txt` / `<AMBIENTE>-publish-<timestamp>.txt` (com `Publicado=False/True` e o caminho do backup) — a pasta `deploy/` é ignorada pelo Git. Esses arquivos registram hash, branch, ambiente, servidor, destinos e o caminho do backup; **nunca** a senha de publicação.

Quando o package executa build, além do `BUILD_INFO.txt` o script grava o `version.json` do frontend (`{ "versao": "1.0.0", "commit": "<GitCommit>", "dataUtc": "<DataUtc>" }`) em `frontend/dist/central-conhecimento-actyon/browser/version.json`, alimentando o badge de versão exibido no Guia do Sistema (`/faq-sistema`).

### Garantias

- Divergência de hash aborta a publicação.
- Commit após o package exige novo validate e novo package.
- Mudança de conteúdo não versionado não é detectada pelo hash.

### Limitações

- `BUILD_INFO` não calcula hash de DLLs, bundles ou `deploy/`.
- O script não recusa working tree sujo.
- O script não compara o `GitBranch` durante o publish.
- O smoke não lê `BUILD_INFO` no servidor.

Por isso, `git status --short` vazio antes do package é obrigatório.

