---
id: 06-inventario-de-entidades-e-tabelas.10
titulo: "Trilha de auditoria — duas vias"
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
tokens_estimados: 302
ficha: docs/modulos/database-explorer.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Trilha de auditoria — duas vias

| Destino | Ações | Quem escreve |
|---|---|---|
| `tbauditoriaimplantacao` (`AUD_`) | `CREATE`, `UPDATE`, `DELETE` com `AntesJson`/`DepoisJson` | Módulo Acessos: **pela fila**. Módulo Implantação: pelo `AuditoriaImplantacaoService`, **inline**. São duas vias independentes que coexistem; o Acessos não reusa o service do Implantação. |
| `tbauditoriaacesso` | `REVEAL_SENHA` | Módulo Acessos, pela fila. Reaproveita `EnderecoIP`/`Navegador`/`Empresa`/`TipoInformacao` que já existiam, mais as 3 colunas `AAC_` que amarravam o evento ao acesso revelado. **Nunca grava a senha.** |

`ProcessadorAuditoria` trunca os valores para caber nas colunas legadas (`Usuario` 15, `Empresa`/`TipoInformacao` 50, `EnderecoIP` 50, `Navegador` 200, `Observacao` 500) e usa `AAC_*` como vínculo real, porque `Empresa` é `varchar(200) NOT NULL` e não aceita nulos.

