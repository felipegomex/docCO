---
id: 06-inventario-de-entidades-e-tabelas.12
titulo: "Modelos que não são tabelas"
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
tokens_estimados: 806
ficha: docs/modulos/database-explorer.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### Modelos que não são tabelas

`ChatPerguntarRequest`/`ChatPerguntarResponse`/`ChatFonteResponse`, os DTOs do catálogo de agentes (`ChatAgenteResponse`/`ChatAgentesResponse`) e os DTOs de Conhecimento são contratos HTTP, não têm `[Table]` nem `DbSet` — os agentes do chat vivem na tabela `tbchat_agente` (o `Workspace` não é exposto ao navegador); as auditorias e configurações do módulo, listadas no inventário, são persistidas. As classes estáticas de domínio também não são tabelas. O antigo `EmpresaAcesso` (projeção do Google Sheets), junto com `EmpresaResumo` e `EmpresaDetalheResponse`, **foi removido**: o módulo Acessos Empresas é persistido, e `AcessoBase` + as 7 entidades do módulo são as únicas formas de guardar esses dados.

As classes estáticas de domínio do Dashboard Integrado ficam em [`ConstantesDashboard.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs) e também não são tabelas:

| Classe | Conteúdo |
|---|---|
| `EtapasComercial`, `StatusOrcamento`, `TiposCusto`, `OrigensLead`, `TiposContatoLead`, `RespostasProposta` | Listas fechadas de valores válidos, com `Validas`/`Validos` e o predicado `EhValido`/`EhValida`; qualquer valor fora da lista é rejeitado com `400`. |
| `FlagsDashboard` | **Interruptor de exibição.** `FaturamentoLiberado` (`const bool = false`) desliga todo valor de faturamento e atraso derivado de `tbcontrato`: a API devolve `0`/lista vazia em vez de publicar uma **projeção** `QTDE_LICENCA × VALOR_UNITARIO`, porque o legado não tem valor total nem data de faturamento e `EhAtivo` inclui `SITUACAO = 'Entregue'` (status de entrega, não de cobrança). Está desligado por decisão do usuário — a informação ainda não foi liberada. **Para religar, trocar `false` → `true`; do lado da API nenhuma outra mudança é necessária** (sem migration, sem DTO, sem rota — os campos continuam no JSON). Os cards/tabelas que exibiam esses valores foram removidos dos templates Angular e precisam ser restaurados junto. Reajustes (datas), custos e orçamentos (`tbcusto`/`tborcamento`) **não** são afetados. Detalhes em [`03-REGRAS-NEGOCIO.md`](../../../referencia/regras-negocio/INDEX.md). |

As classes estáticas do módulo Acessos ficam em [`ConstantesAcessos.cs`](../../../../backend/Central_BackEnd/Models/Acessos/ConstantesAcessos.cs) e seguem o mesmo padrão de "constantes em vez de enum" do projeto:

| Classe | Conteúdo |
|---|---|
| `TiposVpn`, `AmbientesAcesso` | Listas fechadas (`FortClient`/`OpenVPN`/`WireGuard`; `PROD`/`HML`/`DEV`) com `Validos` e `EhValido`. São `const string`, **não enums C#**, para o valor gravado no banco ser legível. |
| `TiposAcessoEmpresa` | Os 5 tipos (`RDP`, `VPN`, `ANYDESK`, `BANCO`, `ACTYONWEB`), usados nas abas e no valor de `AAC_AcessoTipo`. |
| `AcoesAuditoria` | `CREATE`, `UPDATE`, `DELETE` e `REVEAL_SENHA` — este último vai para `tbauditoriaacesso`, não para `tbauditoriaimplantacao`. |

