---
id: sec.08-coluna-gerida-tblead-lead-contatovisualizado
titulo: "Coluna gerida: `tblead.LEAD_ContatoVisualizado`"
tipo: referencia
area: dados
sistema: central
tags: [schema, banco, tabelas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 294
ficha: docs/modulos/dashboard.md
---
> Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Coluna gerida: `tblead.LEAD_ContatoVisualizado`

| Coluna | Tipo | Nulidade | Default | Origem |
|---|---|---|---|---|
| `LEAD_ContatoVisualizado` | `bit` | `NOT NULL` | `0` | Migration `20260928182047_LeadContatoVisualizado` |

Flag **manual** de "o cliente viu o contato", gravada pelo responsável a partir do card do Kanban ou do modal do lead. Não é preenchida por nenhum evento automático: e-mail enviado, contato registrado ou resposta recebida não alteram a coluna. O `Down` da migration remove a coluna. Script idempotente de produção em [Sql/LeadContatoVisualizado_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/LeadContatoVisualizado_Idempotente.sql) (`ProductVersion` `8.0.31`, igual ao das demais); **nenhuma tabela legada é tocada** por esta migration.

