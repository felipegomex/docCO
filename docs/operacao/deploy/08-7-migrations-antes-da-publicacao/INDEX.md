---
id: 08-7-migrations-antes-da-publicacao.indice
titulo: "7. Migrations antes da publicação"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, migration, iis, schema]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy/deploy.ps1
  - scripts/preflight-schema.ps1
  - backend/Central_BackEnd/Migrations
testes:
  - docs/
tokens_estimados: 300
ficha: docs/modulos/database-explorer.md
---

# 7. Migrations antes da publicação

> Índice do documento dividido. Documento original: [08-7-migrations-antes-da-publicacao.md](../08-7-migrations-antes-da-publicacao.md).
> Ficha: [Database Explorer](../../../modulos/database-explorer.md).

> Parte de [docs/operacao/deploy.md](../../deploy.md).


O pipeline obrigatório não pode ignorar migration quando o commit altera o modelo.

## Partes

- [Regras](./01-regras.md)
- [Gate](./02-gate.md)
- [Gate — parte 2](./03-gate-parte-2.md)
- [Migration do módulo Acessos Empresas: `20260928212856_AcessosEmpresas`](./04-migration-do-modulo-acessos-empresas-20260928212856-acessose.md)
- [Migration aplicada: `20260929034944_KanbanGeral_SetoresEtapasBloqueios` (Kanban Geral)](./05-migration-aplicada-20260929034944-kanbangeral-setoresetapasb.md)
- [`20260929034944_KanbanGeral_SetoresEtapasBloqueios` em detalhe](./06-20260929034944-kanbangeral-setoresetapasbloqueios-em-detalhe.md)
- [Migration do eixo Setor → Função: `20260929221556_SetorParaFuncao`](./07-migration-do-eixo-setor-funcao-20260929221556-setorparafunca.md)
- [Migration `20260930132443_AcessosGrupoClienteErp` (grupos do Acessos via ERP) — aplicada em 2026-09-30](./08-migration-20260930132443-acessosgrupoclienteerp-grupos-do-ac.md)
- [Migration pendente no banco de produção: `20260928182047_LeadContatoVisualizado`](./09-migration-pendente-no-banco-de-producao-20260928182047-leadc.md)
