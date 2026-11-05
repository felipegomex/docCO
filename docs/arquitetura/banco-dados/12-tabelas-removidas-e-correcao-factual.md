---
id: sec.12-tabelas-removidas-e-correcao-factual
titulo: "Tabelas removidas e correção factual"
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
tokens_estimados: 339
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Tabelas removidas e correção factual

| Removida | Motivo |
|---|---|
| `IMPL_Cliente` | Entidade `Cliente` morta: nenhum serviço ou controller a usava. O fonte de clientes é a legada `tbcliente`. Entidade, `DbSet<Cliente>` e o bloco de mapeamento foram apagados. |
| `CC_Funcao` | Substituída pela legada `tbfuncao` enriquecida. |
| `AgendaEvento` e `AgendaEventoParticipante` | Órfãs do rollback da Agenda, 0 linhas, com FK ativa para o que era `IMPL_Projeto`. |

> ⚠️ **Correção factual:** documentos antigos afirmavam que as órfãs da Agenda eram `CC_Agenda`, `CC_Perfil` e `CC_Auditoria`. **Isso estava errado.** As tabelas órfãs reais eram `AgendaEvento` e `AgendaEventoParticipante`, e ambas foram removidas. `CC_TipoEvento` e `CC_AgendaParticipante` nunca foram órfãs — foram apenas renomeadas para `tbtipoevento` e `tbagendaparticipante` e continuam em uso pela Agenda. A afirmação antiga estava nos documentos de projeto da Agenda, removidos do repositório.

