---
id: 06-5-modelos-dbcontext-e-relacionamentos.04
titulo: "5.3 Relações críticas"
tipo: referencia
area: backend
sistema: central
tags: [backend, modelos, dbcontext, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 842
ficha: docs/modulos/database-explorer.md
---
> Parte de [5. Modelos, DbContext e relacionamentos](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 5.3 Relações críticas

- Projeto → TipoProjeto usa `Restrict`; Projeto → Cliente legado usa `SetNull`.
- Projeto → cards usa `Cascade`; índice único garante uma linha por `(ProjetoId, Ordem)`.
- Checklist, documentos, histórico e comentários de card usam `Cascade`.
- Tarefa → Projeto usa `Cascade`; Tarefa → card de Projeto usa `Restrict`; coluna do Kanban usa `SetNull`.
- **Kanban Geral:** Tarefa, Projeto e Coluna do Kanban → `Funcao` usam `SetNull` (perder a função não apaga ninguém: ela classifica o quadro, não é dona do dado); `EtapaFuncao` → `Funcao` usa `Cascade`; Tarefa → `EtapaFuncao` é `NoAction` no `Down` da `20260929221556_SetorParaFuncao`. A visão do setor (`ColunaFuncao`) → `ColunaKanban` e → `Funcao` usam **`Cascade`**: a coluna do catálogo e a função donas da visão derrubam a visão junto.
- **RBAC:** `Operador` → `Cargo` usa `SetNull` (perder o cargo não apaga o operador); `PermissaoCargo` → `Cargo`/`Recurso`/`Funcao` e `PermissaoOperador` → `Operador`/`Recurso`/`Funcao` usam **`Restrict`** — a matriz e a exceção não são removidas em cascata. `PEC_FuncaoId`/`PEO_FuncaoId` são anuláveis (`NULL` = todas as funções).
- Tarefa–responsável, Tarefa–chamado e apontamentos usam chaves compostas/chave própria e `Cascade` a partir da Tarefa.
- Agenda → Projeto e TipoEvento usam `SetNull`; Agenda → participantes usa `Cascade`; participante → Operador usa `Restrict`.
- Lead → Cliente legado e Lead → Operador usam `SetNull`; Oportunidade → Lead usa `Cascade`; Oportunidade → Produto legado usa `SetNull` lógico (sem FK física, tabela fora das migrations); Orçamento → Cliente legado usa `SetNull`; Custos não têm FK.
- Cliente legado → grupo econômico usa `Restrict` (FK não declarada no legado, então explícita para leitura) e Cliente legado ↔ Contrato legado usa `Restrict`.
- **Acessos Empresas:** `ClienteAcesso` → grupo econômico usa `Restrict` (FK anulável para `tbcliente_grupo.GRUPO_ID`, fonte única de grupos) e `ClienteAcesso` → `tbcliente.CLIENTE_ID` (`CAO_ClienteId`, anulável, `Restrict` — empilha a unidade sobre a empresa do ERP); as 5 tabelas de acesso → `ClienteAcesso` também, com `ClienteAcessoId` obrigatório. Nada é removido em cascata — o SQL Server recusa múltiplos caminhos e o projeto já usa `Restrict` pelo mesmo motivo em `tbprojeto`/`tblead`; quem desativa é a coluna `Ativo`.
- As cinco tabelas legadas (`tbcliente`, `tbcliente_grupo`, `tbchamado`, `tbfuncionario`, `tbcontrato`) são marcadas com `ExcludeFromMigrations()`.
- **Atendimento WhatsApp:** todas as FKs do módulo usam `Restrict` (`ON DELETE NO ACTION` no banco) — conversa → número, contato e atendimento; atendimento → conversa, número e `tboperador`; mensagem → atendimento, conversa e `tboperador`; evento → atendimento e `tboperador`. A FK entre conversa e atendimento é **circular** e entra por `ALTER TABLE` depois que as duas tabelas existem. O módulo **não apaga fisicamente**: desativar o número preserva atendimentos, mensagens e eventos. A `tbwhatsapp_webhook_fila` **não tem FK** — é fila, não entidade de domínio.

