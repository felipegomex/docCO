---
id: estrutura-projeto.03
titulo: "3. Documentação canônica"
tipo: arquitetura
area: processo
sistema: central
tags: [monorepo, convencoes, pipeline, agents]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - AGENTS.md
  - scripts/validate.ps1
testes:
  - docs/
tokens_estimados: 1100
ficha: docs/modulos/kanban.md
---
> Parte de [00 — Estrutura, leitura e fluxo de trabalho](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## 3. Documentação canônica

O conjunto canônico possui **12 documentos** (este README + os onze arquivos `00` a `10` da matriz de [`README.md`](../../README.md)), organizados em três subpastas de `docs/` — `arquitetura/`, `referencia/` e `operacao/` — desde a reestruturação de 2026-10-17:

- **Entrada e fundação:** `docs/README.md`, `arquitetura/estrutura-projeto.md`, `arquitetura/visao-geral.md` e `arquitetura/componentes.md`.
- **Domínio e dados:** [`referencia/regras-negocio.md`](../../referencia/regras-negocio/INDEX.md) e [`arquitetura/banco-dados.md`](../banco-dados.md).
- **Implementação e QA:** [`referencia/endpoints.md`](../../referencia/endpoints.md), [`referencia/frontend.md`](../../referencia/frontend.md), [`referencia/backend.md`](../../referencia/backend.md) e [`referencia/qa-historias.md`](../../referencia/qa-historias.md).
- **Operação:** [`operacao/troubleshooting.md`](../../operacao/troubleshooting.md) e [`operacao/deploy.md`](../../operacao/deploy.md).

A superfície HTTP está em [`referencia/endpoints.md`](../../referencia/endpoints.md); o frontend, em [`referencia/frontend.md`](../../referencia/frontend.md); o backend, em [`referencia/backend.md`](../../referencia/backend.md); e os cenários de QA, em [`referencia/qa-historias.md`](../../referencia/qa-historias.md).

### A auditoria do backend saiu do conjunto canônico

`11-AUDITORIA-ARQUITETURA-BACKEND.md` foi movido para [`material-de-projeto/backup/11-AUDITORIA-ARQUITETURA-BACKEND.md`](../../../material-de-projeto/backup/11-AUDITORIA-ARQUITETURA-BACKEND.md) em 2026-10-17 e **não integra a matriz**. O texto continua valendo: ele tem `status: proposta` e é um diagnóstico datado mais um plano de melhoria incremental, produzido sem alteração de código (as Etapas 0 a 4 continuam não executadas). Os canônicos de `00` a `10` **descrevem o que existe**; o diagnóstico **não**. Quando os dois divergirem, o documento temático (`componentes.md`, `banco-dados.md`, `endpoints.md`, `backend.md`) prevalece, e a divergência deve ser corrigida no temático — não no diagnóstico. Confirme cada afirmação da auditoria no código antes de agir sobre ela.

### Regras canônicas

- Um assunto por arquivo; não transformar um documento histórico em uma lista de novos capítulos.
- Atualizar é substituir o trecho obsoleto, não anexar conteúdo sem remover o anterior.
- Manter cerca de 600 linhas como limite por volume; subdividir antes de crescer sem função.
- Links devem apontar para arquivos, nunca para âncoras profundas.
- Nomes de rotas, cookies, flags, classes, pastas e URLs devem ser conferidos no código antes da publicação.
- Não registrar senhas, chaves, tokens, credenciais ou connection strings com credenciais.
- Ao mudar apenas documentação, não alterar código, automações ou arquivos de configuração.

### Segredos: onde eles ficam

Nenhum arquivo versionado contém senha, API key ou connection string com credencial. A regra completa está em [`AGENTS.md`](../../../AGENTS.md) (seção `§ Segredos`); o resumo operacional:

| Tipo | Onde |
|---|---|
| Senha do banco | `dotnet user-secrets` ou variável de ambiente |
| Chave de API (IA, Sheets, RAG) | `dotnet user-secrets` ou variável de ambiente |
| Chave de JWT / AES | variável de ambiente no servidor (`JWT_KEY`, `Acessos__ChaveCifra`) |
| Credencial de IIS no deploy | parâmetro explícito (`-SenhaRemota`), nunca constante no script |

**Quem garante:** [`../scripts/check-secrets.ps1`](../../../scripts/check-secrets.ps1), executado como **passo 1/5** do [`../scripts/validate.ps1`](../../../scripts/validate.ps1). Ele varre o **índice do Git** — o que realmente entraria no commit — e bloqueia o gate.

**Precedentes que motivaram a regra:** `scripts/deploy/deploy.ps1` deixou de ter senha fixa (linhas 8-9) e passou a exigir `-SenhaRemota`; em 2026-10-11 dois arquivos com credencial do banco foram removidos (`archive/_inspect-colunas.ps1` e `scripts/importar_acessos_xlsx.py`). Roteiro de rotação e o que resolve ou não resolve: [`09-TROUBLESHOOTING.md`](../../operacao/troubleshooting.md), seção "Segredos no repositório".

