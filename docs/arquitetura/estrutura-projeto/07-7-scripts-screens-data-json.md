---
id: estrutura-projeto.07
titulo: "7. `scripts/screens-data.json`"
tipo: arquitetura
area: processo
sistema: central
tags: [monorepo, convencoes, pipeline, agents]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - AGENTS.md
  - scripts/validate.ps1
testes:
  - docs/
tokens_estimados: 503
ficha: docs/modulos/README.md
---
> Parte de [00 — Estrutura, leitura e fluxo de trabalho](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 7. `scripts/screens-data.json`

O arquivo `scripts/screens-data.json` é um **inventário auxiliar gerado** por `npx tsx scripts/extract-screens.ts`, usado para localizar candidatos e comparar uma extração anterior.

Ele não é fonte factual porque:

- a extração é textual e pode omitir rotas eager, métodos não `async` ou formatos de controller não cobertos pela expressão regular;
- pode conter redirecionamentos, componentes antigos ou referências incompletas;
- o inventário foi regerado e já inclui as telas do Dashboard Integrado e do módulo Acessos Empresas; mesmo assim, pode ficar datado — verifique a data do arquivo antes de usá-lo;
- o inventário lista rotas e endpoints como extraídos por expressão regular — confirme nomes, caminhos e módulos no código antes de usar;
- não substitui `app.routes.ts`, `features.routes.ts`, `*.routes.ts` e os controllers;
- não descreve autorização, corpos de requisição ou comportamento de runtime.

A geração atual remove timestamp variável e normaliza caminhos para relativos ao repositório. Regere o arquivo com o script antes de comparar; **não edite `screens-data.json` à mão**, porque a próxima geração sobrescreve o ajuste.

Use-o como pista. Confirme toda informação no código antes de documentar, indexar ou planejar uma alteração.

> O workflow `.github/workflows/docs-sync.yml` valida **somente** o inventário auxiliar `scripts/screens-data.json`. O texto canônico é revisado pelo `docs-writer` na conversa.

