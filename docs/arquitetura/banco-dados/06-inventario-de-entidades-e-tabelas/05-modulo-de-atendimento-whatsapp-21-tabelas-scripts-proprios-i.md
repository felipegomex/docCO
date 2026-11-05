---
id: 06-inventario-de-entidades-e-tabelas.05
titulo: "Módulo de atendimento WhatsApp — 22 tabelas, scripts próprios — introdução — parte 2"
tipo: referencia
area: dados
sistema: central
tags: [schema, tabelas, entidades, sqlserver]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
  - backend/Central_BackEnd/Migrations
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 308
ficha: docs/modulos/whatsapp.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### Módulo de atendimento WhatsApp — 22 tabelas, scripts próprios — introdução — parte 2

⚠️ **`ON DELETE RESTRICT` não é sintaxe válida em TSQL** — aborta o `CREATE TABLE` com erro apontando a linha da constraint, não o `ON DELETE`. Use `NO ACTION` (padrão do SQL Server, equivalente ao `RESTRICT` do padrão SQL). No C#, `DeleteBehavior.Restrict`.

**FK circular.** `tbwhatsapp_conversa.WAT_AtendimentoId` → `tbwhatsapp_atendimento` só pode ser criada **depois** que o atendimento existir: a tabela da conversa nasce sem a FK e a constraint entra por `ALTER TABLE` (idempotente por `sys.foreign_keys`). Como ela é `NO ACTION`, a **exclusão forçada** do número (`?forcar=true`) **anula `Conversa.AtendimentoId` em memória** antes de remover os atendimentos da cascata — sem isso o SQL Server recusa o `DELETE` do atendimento com violação de FK (causa do incidente de 2026-10-26, corrigido no `RemoverAsync`).

