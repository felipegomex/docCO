---
id: 06-inventario-de-entidades-e-tabelas.03
titulo: "Módulo de Ideias — seis tabelas, quatro índices únicos filtrados"
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
tokens_estimados: 659
ficha: docs/modulos/ideias.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Ideias, Melhorias e Erros](../../../modulos/ideias.md).

### Módulo de Ideias — seis tabelas, quatro índices únicos filtrados

Criado em 2026-10-11 (fase F1). Script único: [`Migrations/Sql/Ideias_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/Ideias_Idempotente.sql).

**Sem coluna JSON para os campos por tipo.** A especificação original propunha `IDE_CamposEspecificosJson`. Foi descartado: os três tipos têm formulário fixo e os filtros da listagem são por tipo, status e período — colunas dedicated atendem sem `OPENJSON`, que o banco legado (`compatibility_level 100`) não tem.

**Quatro índices únicos filtrados.** Cada um resolve um problema específico que o código sozinho não garante:

| Índice | Filtro | Garante |
|---|---|---|
| `UX_tbideia_IDE_TarefaVinculadaId` | `IDE_TarefaVinculadaId IS NOT NULL` | O mesmo cartão do Kanban não pode ser vinculado a duas solicitações. É a última linha de defesa: mesmo que o serviço tenha uma janela de corrida entre checar e gravar, o banco recusa a segunda. |
| `UX_tbideia_IDE_DemoChave` | `IDE_DemoChave IS NOT NULL` | Idempotência do seed de demonstração: rodar duas vezes não duplica. |
| `UX_tbgestor_Funcao_Operador` | `GES_Ativo = 1` | Um operador não pode constar duas vezes na mesma função. Filtrado por ativo para que inativar e religar não conflite com o histórico. |
| `UX_tbgestor_Geral` | `GES_Ativo = 1 AND GES_FuncaoId IS NULL` | No máximo um gestor geral ativo. O `IS NULL` é o que separa o gestor geral dos por-função: sem ele, `NULL` colidiria com `NULL` e nenhuma função poderia ter gestor. |

**`ON DELETE RESTRICT` em todas as FKs do módulo**, pelo mesmo motivo já valado no módulo Acessos (o SQL Server recusa múltiplos caminhos em cascata) e porque o módulo **não apaga fisicamente**: o arquivamento é lógico e o histórico de negócio precisa sobreviver.

**`CHECK` constraints** nos enums gravados como `tinyint`: `IDE_Tipo` 0–2, `IDE_Status` 0–6, `IDE_PrioridadeSugerida` 0–2, `IDE_PrioridadeOficial` anulável 0–2, `MSG_Tipo` 0–1, `AID_Estado` 0–6, `ESP_Versao ≥ 1`, `ESP_Estado` 0–2. Um valor fora da faixa é recusado pelo banco, não apenas pelo código.

**Estado em 2026-10-11:** script escrito e revisado, **não aplicado**. Módulo sem service, sem controller e sem tela. Cenários do módulo em [`referencia/qa-historias.md`](../../../referencia/qa-historias.md) §10.

