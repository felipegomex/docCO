---
id: sec.03-contagens-confirmadas
titulo: "Contagens confirmadas"
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
tokens_estimados: 1727
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Contagens confirmadas

| Medida | Total | Observação |
|---|---:|---|
| `DbSet` em `AppDbContext` | 81 | Inclui seis espelhos legados de leitura, as cinco entidades de Chat/Conhecimento, os **vinte e um** conjuntos do módulo de WhatsApp e os **quatro** do RBAC (`Cargos`, `PermissoesRecurso`, `PermissoesCargo`, `PermissoesOperador`); os demais conjuntos estão discriminados no inventário abaixo. |
| Tabelas gerenciadas pelas migrações | 47 | As seis legadas usam `ExcludeFromMigrations()`. As cinco tabelas `tbchat_*`/`tbconhecimento_*` são criadas pela migration `20261021003310_ChatConhecimento`; tabelas de Ideias e `tbdoc_doc` seguem as formas indicadas nas respectivas seções. |
| Tabelas do RBAC (script próprio, **sem migration EF**) | 4 | `tbcargo`, `tbpermissao_recurso`, `tbperm_cargo` e `tbperm_operador`, criadas por `Permissoes_Idempotente.sql`; não estão no snapshot do EF. |
| Tabelas do módulo de WhatsApp (scripts próprios, **sem migration EF**) | 22 | As 7 de negócio (`tbwhatsapp_numero`, `_contato`, `_conversa`, `_atendimento`, `_mensagem`, `_evento` e `_webhook_fila`) vêm de [`Sql/WhatsApp_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Idempotente.sql); as fases 0/1/3/4/5 acrescentam `_parametro`, `_resposta_rapida`, `_sla_config`/`_sla_horario`/`_sla_historico` e `_retencao_config`/`_retencao_execucao`. **Fase D (categorias/tags) foi removida** em 2026-10-23; as tabelas `_categoria`, `_tag`, `_atendimento_tag` são dropadas por `WhatsApp_RemoveTaxonomia_Idempotente.sql`. **Fase C** acrescenta `_grupo` e `_participante`. **Fase F** acrescenta `_auditoria`. **Fase G** acrescenta `_ia_config`. **Fase 06 do novo ciclo** acrescenta `_etiqueta` e `_atendimento_etiqueta`. **A Fase 2 não criou tabela.** **A Fase 09 também não cria tabela, coluna, índice, migration ou script**: o hub `/hubs/whatsapp` e o indicador de digitação são notificação em tempo real sobre o schema existente. **A F6 (Visto) também não cria tabela nem coluna**: `WhatsApp_Visto_Idempotente.sql` só recria a `CHECK` do status para 0..6. **A Fase 3 (foto do contato) não cria schema:** usa `WAT_FotoUrl`/`WAT_FotoAtualizadaEm` do script das Fases 0/1. **A Fase 8 (reações) acrescenta `_mensagem_reacao`** por [`WhatsApp_Fase9_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase9_Idempotente.sql) — **aplicado em 2026-10-30** em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML` (`192.168.2.154`) com confirmação de tabela, `WAT_Emoji` e índice único; o preflight de deploy deixou de bloquear nesses bancos. |
| Tabelas legadas somente leitura | 6 | `tbcliente`, `tbcliente_grupo`, `tbchamado`, `tbfuncionario`, `tbcontrato` e `tbproduto`. |
| Classes de migração presentes | 36 | A mais recente é `20261030170000_ConfiguracaoSistema` (as 4 mais novas em relação a 2026-10-29: `AgendaSerieId`, `ChatAgentes`, `ChatProvedorConfig`, `ConfiguracaoSistema`). A presença do arquivo não prova que foi aplicada no banco-alvo. O módulo de WhatsApp **não** tem classe de migração. |
| Operações de controller | 303 | Recalculada a partir dos atributos `[Http*]` (recontagem 2026-11-05); o catálogo detalha a distribuição atual, incluindo os controllers de Chat (**1** operação — `GET /api/chat/sso`), Conhecimento (18), WhatsApp (15 classes, **79 operações** — a Fase 09 não acrescentou rota REST; a F6 acrescentou `POST /atendimentos/{id}/visto`, a Fase 3, `GET /contatos/{id}/foto`, a Fase 8, `POST .../mensagens/{mensagemId}/reacao`, e a rodada 2026-11-03, `sem-contato`/`vincular`/`foto?chatId=` e `finalizacao-automatica`), Permissões (5), Cargos e as 6 operações de Operadores. O hub SignalR `/hubs/whatsapp` **não** entra nessa contagem. |
| DTOs de acesso persistidos no `AppDbContext` | 0 | Requests/responses do Chat e Conhecimento são contratos; persistem as entidades de workspace, auditoria, revisores e credenciais Gitea. |

### O snapshot do EF — estado da sincronia e por que `dotnet ef migrations add` segue perigoso aqui

**Não rode `dotnet ef migrations add` neste repositório.** Já foi tentado em 2026-10-11 para o módulo de Ideias e o resultado foi **descartado por ser destrutivo**.

**Causa histórica.** As migrations `20261008090000_ComercialFunilProdutos` e `20261014000000_RemoveClkBloqueada` foram escritas à mão, **sem o `.Designer.cs`** (a segunda sincronizou o `AppDbContextModelSnapshot.cs` à mão, removendo a propriedade). O snapshot chegou a divergir do modelo/banco nas duas direções:

| No snapshot, mas **não** no modelo C# | No modelo/banco, mas **não** no snapshot |
|---|---|
| `tboperador.DATA_ULTIMO_ACESSO` | `tblead.LEAD_Arquivado`, `LEAD_DataArquivamento`, `LEAD_MotivoArquivamento` |
| `tboperador.DATA_FECHAMENTO` | `tbleadcontato.LDC_NomeContato` |
| `tboperador.VALIDATED_AT` | `tboportunidade.OPP_ProdutoId`, `OPP_QtdeLicencas`, `OPP_ValorImplementacao`, `OPP_ValorPorLicenca`, `OPP_MotivoPerda` |

**Sintoma observado.** Com o snapshot divergente, `dotnet ef migrations add IdeiasGestao` gerou um `Up()` com operações de **três tabelas que nada tinham a ver com o módulo**, entre elas:

```csharp
migrationBuilder.DropColumn(name: "DATA_ULTIMO_ACESSO", table: "tboperador");
```

Aplicar essa migration em `dbBUSINESS_HML` teria **destruído a coluna `DATA_ULTIMO_ACESSO` da tabela de operadores** e tentado recriar as 9 colunas do funil comercial, que já existem.

**Como o módulo contornou.** A migration gerada foi removida com `dotnet ef migrations remove` e o `AppDbContextModelSnapshot.cs` foi restaurado ao estado do HEAD (o EF 8 reintroduz apenas ruído cosmético, `ToTable("x", (string)null)`, descartado). O esquema das 6 tabelas do módulo vive **apenas** em [`Migrations/Sql/Ideias_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/Ideias_Idempotente.sql), escrito à mão e 100% aditivo — sem `DROP`, sem `ALTER COLUMN`, sem `DELETE`.

**Estado atual (2026-10-19).** A fase 1 do plano docs-api (`TbDocDocumentos`, escrita à mão sem `.Designer.cs`) **sincronizou o snapshot com o modelo inteiro**: hoje ele descreve o funil comercial (`LEAD_Arquivado`, `LDC_NomeContato`, `OPP_*`), as 6 entidades de Ideias e `Gestores` que faltavam e a nova `tbdoc_doc`. A divergência snapshot × modelo acima está resolvida; a regra operacional permanece — qualquer migration gerada pelo EF precisa ser **revisada linha a linha** antes de virar script, porque o snapshot pode voltar a divergir sem aviso quando outra migration for escrita à mão. Diagnóstico original em [`11-AUDITORIA-ARQUITETURA-BACKEND.md`](../../../material-de-projeto/backup/11-AUDITORIA-ARQUITETURA-BACKEND.md) (seção H.5.2).

### Fotografia das legadas (homologação, 2026-09-28)

Contagens observadas no `dbBUSINESS_HML`, úteis como referência de volume e de sanidade dos LEFT JOINs. **Não são contrato**: reconfirme no banco alvo antes de usar em decisão.

| Medida | Valor |
|---|---:|
| Clientes em `tbcliente` | 154 |
| Contratos em `tbcontrato` | 185 |
| Soma de `QTDE_LICENCA` | 2021 |
| Clientes com `GRUPO_ID` preenchido | 18 |
| Grupos em `tbcliente_grupo` | 5 |
| Clientes sem nenhum contrato | 6 |

Distribuição de `tbcontrato.SITUACAO`: `NULL` 72, `Entregue` 66, `Pendente` 47 — os três únicos valores observados. `tbcliente.ATIVO` usa somente `S`/`N`.

