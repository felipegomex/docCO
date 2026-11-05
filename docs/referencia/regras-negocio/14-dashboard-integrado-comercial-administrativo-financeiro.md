---
id: regras-negocio.14
titulo: "Dashboard Integrado (comercial, administrativo, financeiro)"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2869
ficha: docs/modulos/dashboard.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

## Dashboard Integrado (comercial, administrativo, financeiro)

**Situação: implementada; funil comercial, leitura administrativa e escrita financeira.**

- O funil comercial usa as etapas `Prospecção`, `Apresentação`, `Proposta` e `Negócio Fechado` (constantes em `ConstantesDashboard.cs`); qualquer outra etapa é rejeitada com `400`.
- Mover um lead para `Negócio Fechado` **não cria contrato**: a automação está adiada por decisão de produto e o movimento apenas grava `UltimoContato`.
- Excluir um lead apaga em cascata as suas oportunidades; as FKs para `tbcliente`/`tboperador` são `SetNull`, então apagar o legado não derruba o lead.
- Enviar proposta (`PATCH /oportunidades/{id}/proposta` com `dataPropostaEnviada`) move a oportunidade para a etapa `Proposta`; limpar `respostaCliente` zera também `dataResposta` e `motivoPerda`.
- **A tela de Oportunidades foi removida** por ser redundante com a prospecção. A entidade `tboportunidade` e seus endpoints foram **mantidos**: a aba Propostas consome `OportunidadeResumo`, o KPI comercial (`ValorPipeline`/`ValorFechado`) e o campo `valorOportunidades` do card somam `tboportunidade`. Como a tela removida era a única forma de criar oportunidade, a criação foi movida para o modal do lead (visível apenas na edição de um lead existente), e o empty state de Propostas aponta para o Kanban.
- **Oportunidades só na Proposta**: `POST /oportunidades` exige lead com `StatusEtapa == "Proposta"` e não arquivado (`400` caso contrário). Cada produto marcado no checklist gera **1 linha em `tboportunidade`** com `ProdutoId` (`tbproduto`), `ValorImplementacao`, `ValorPorLicenca` (**PA e licença são o mesmo conceito**) e `QtdeLicencas`; `ValorEstimado = implementação + porLicença × qtde`. A observação por produto usa a coluna `OPP_Observacao` existente.
- **Arquivamento**: negócios fechados e propostas não convertidas saem da visão principal via `LEAD_Arquivado` (soft-delete com `DataArquivamento`/`MotivoArquivamento`), preservando o histórico. Listagem e KPI excluem arquivados por padrão (`incluirArquivados`/`somenteArquivados`).
- **Proposta não convertida**: `respostaCliente = "NÃO"` + `motivoPerda` na oportunidade, combinada com o arquivamento do lead. O motivo identifica clientes que receberam proposta mas não fecharam.
- O módulo administrativo é somente leitura sobre as legadas `tbcliente`, `tbcliente_grupo` e `tbcontrato`; nenhum endpoint do módulo altera dados legados. `diasParaReajuste` é calculado de `DataProximoReajuste` e é **anulável**: contrato sem data de reajuste devolve `null`, nunca `0`.
- Custos aceitam apenas os tipos `Salário`, `Imposto`, `Estrutura` e `Outro`, com competência no formato `AAAA-MM`; orçamentos validam status entre `Rascunho`, `Enviado`, `Aprovado` e `Faturado`.
- Transições de orçamento: entrar em `Aprovado` grava `DataAprovacao`, sair de `Aprovado` limpa e `Faturado` grava `DateTime.Now`; `dataFaturamento` enviada no `PUT` é aceita como informada.
- Os três módulos usam a política `validacao` no rate limit; erros de argumento voltam como `400 { mensagem }` (e não como 500).

### "Cliente viu o contato" — flag manual

**Situação: implementada; não é inferida de evento algum, é marcada pelo responsável.**

- O campo é `tblead.LEAD_ContatoVisualizado bit NOT NULL DEFAULT 0` (`TbLead.ContatoVisualizado`), gravado pela migration `20260928182047_LeadContatoVisualizado`.
- Semântica: o responsável **afirma manualmente** que o cliente visualizou o contato. O sistema **não** deriva esse estado de e-mail enviado, resposta registrada ou qualquer evento automático; nada no código alterna a flag sozinho.
- Toggle no card do Kanban (`PATCH /api/v1/dashboard/comercial/leads/{id}/contato-visualizado`, corpo `{ "visualizado": bool }`) e checkbox no modal do lead (`PUT /leads/{id}` com `contatoVisualizado`, aplicado **somente quando o campo é informado**).
- O endpoint dedicado grava `UsuarioAlteracao`/`DataAlteracao` e **não revalida** origem nem cliente, porque não altera esses campos. Ele é independente do registro de contatos (`POST /leads/{id}/contatos`), que continua sendo a fonte do rastro de prospecção.
- Visual: "Viu" (verde) ou "Não viu" (cinza) no card; no modal, rótulo alterna entre "Cliente visualizou o contato" e "Cliente ainda não visualizou o contato".
- A flag é **independente** do badge "aguardando resposta" (que é derivado do último tipo de contato ser `Envio de mensagem`); os dois podem coexistir em estados contraditórios porque não se auto-sincronizam.

### Regra de contrato vigente no legado

**Situação: implementada e centralizada em `TbContratoLegado.EhAtivo`.**

- Um contrato conta como vigente quando `ATIVO == 'S'` **ou** `SITUACAO == 'Entregue'` (comparação sem diferença de caixa, `OrdinalIgnoreCase`).
- A regra antiga testava `SITUACAO == "Ativo"`, valor que **nunca ocorre** no legado: os valores reais de `tbcontrato.SITUACAO` são `NULL`, `'Entregue'` e `'Pendente'`.
- A propriedade é `[NotMapped]` e substituiu a condição duplicada que existia em `AdministrativoService` e `FinanceiroService`; as duas agora consomem `EhAtivo`.
- `ValorMensal` (`[NotMapped]`) é `QtdeLicenca × ValorUnitario`, com **fallback de 1 licença** quando o legado não informa a quantidade. É a base de `faturamentoMensalContratos`, `FaturamentoPrevistoMes` e do valor dos contratos em atraso — mas só quando `FlagsDashboard.FaturamentoLiberado` estiver ligado; ver “Faturamento não liberado” abaixo.
- O legado **não tem coluna de data de faturamento**: o ciclo é `DIA_FATURAMENTO smallint` (dia do mês). Um contrato do mês corrente está vencido quando `DiaFaturamento` está entre 1 e 31 e é **menor que o dia de hoje**; `DiasAtraso = hoje.Day - DiaFaturamento`. Contrato sem `DiaFaturamento` nunca entra na lista de atrasos — e, com o interruptor desligado, a lista volta vazia de qualquer forma.
- Contrato legado é somente leitura: a criação continua sendo feita no sistema de origem.

### Faturamento não liberado — interruptor de exibição

**Situação: desligado por decisão do usuário. É uma regra de produto, não um bug; não "corrija" reativando sem o contexto abaixo.**

`ConstantesDashboard.FlagsDashboard.FaturamentoLiberado` é `const bool = false` e governa **todo** valor de faturamento e atraso do módulo:

```csharp
public static class FlagsDashboard
{
    public const bool FaturamentoLiberado = false;   // desligado de propósito
}
```

**Por que desligar:** todo valor de faturamento do módulo é uma **projeção** derivada de `tbcontrato` (`QTDE_LICENCA × VALOR_UNITARIO`), não receita confirmada. O legado **não tem coluna de valor total nem de data de faturamento** (só `DIA_FATURAMENTO`, o dia do mês), e a regra de contrato vigente (`TbContratoLegado.EhAtivo`) inclui `SITUACAO = 'Entregue'`, que é status de **entrega**, não de cobrança. A informação ainda não foi liberada pelo Financeiro, então a API devolve **zero** em vez de publicar um número enganoso.

**Efeito com o interruptor desligado** (`false`):

| Ponto | Comportamento desligado | Continua funcionando |
|---|---|---|
| `AdministrativoService.ObterKpiAsync` | `FaturamentoMensalContratos = 0m` | clientes/contratos ativos, licenças, reajustes 90d, leads no funil |
| `AdministrativoService.ListarContratosAsync` | `ContratoResumo.ValorTotal = 0m` | quantidade, valor unitário, `diaFaturamento`, `situacao`, `diasParaReajuste` |
| `FinanceiroService.ObterKpiAsync` | `FaturamentoPrevistoMes`, `FaturamentoAtrasado`, `ResultadoMes` e `ContratosAtrasados` em `0`; o cálculo de `vencidos` é pulado (`new List<TbContratoLegado>()`) | `CustoMes` (de `tbcusto`), `OrcamentosAprovadosPendentes`, `ReajustesProximos90Dias`, `EvolucaoCustos` |
| `FinanceiroService.ListarAtrasosAsync` | retorno antecipado: **lista vazia** | o endpoint existe e volta a responder ao religar |
| Dashboard home | bloco "Financeiro" e cards de Faturamento mensal / Reajustes 90d removidos; o `forkJoin` não chama `/financeiro/kpis` | KPIs comercial e administrativo |
| Tela Administrativa | cards de Faturamento mensal / Reajustes 90d e o alerta "N reajuste(s) em até 30 dias" removidos | 3 KPIs (clientes ativos, contratos ativos, licenças) e a base de clientes |
| Tela Financeiro | KPIs de previsto/atrasado/custo/resultado e a tabela "Faturamentos em atraso" removidos | 2 KPIs (orçamentos aprovados pendentes, reajustes 90d), gráfico de evolução e CRUD de custos e orçamentos |

**O que permanece de propósito:** o gráfico de evolução, a tabela de Custos e a de Orçamentos vêm de `tbcusto`/`tborcamento`, tabelas que o sistema **gerencia e escreve** (ambas vazias hoje). A coluna "Faturamento" dos Orçamentos também permanece porque é `tborcamento.dataFaturamento`, **dado nosso**, e não a projeção do legado. Reajustes (`ReajustesProximos90Dias`) continuam calculados porque são **data**, não dinheiro.

**Como religar:** trocar `false` → `true` em `FlagsDashboard.FaturamentoLiberado`. **Do ponto de vista da API, não há outra mudança necessária** — sem migration, sem ajuste de DTO e sem mudança de rota: os campos continuam no JSON e `GET /financeiro/atrasos` volta a popular. Ressalva: nesta mesma mudança os **cards/tabelas foram removidos dos templates** (home, Administrativo e Financeiro) e `FinanceiroService.listarAtrasos()` ficou sem consumidor na UI; restaurar esses blocos faz parte do religamento.

Números de referência do homolog medidos com o interruptor **ligado**, para reconhecer o valor quando ele voltar: faturamento mensal de **R$ 259.454,08** (125 contratos, 1600 licenças) e faturamento atrasado de **R$ 163.970,03** (68 contratos). Com o interruptor desligado: `/administrativo/resumo` → `clientesAtivos 70`, `contratosAtivos 125`, `licencasTotais 1600`, `faturamentoMensalContratos 0`, `reajustesProximos90Dias 8`, `leadsNoFunil 1`; `/financeiro/kpis` → todos os valores de faturamento/resultado/atraso em `0`, `orcamentosAprovadosPendentes 0`, `reajustesProximos90Dias 8`; `/financeiro/atrasos` → 0 itens; `/administrativo/contratos` → 185 contratos, 101 com `diasParaReajuste` `null` e 1 dentro de 0–30 dias.

### Base de clientes e somatório de licenças

**Situação: implementada; agregação no SQL com LEFT JOIN obrigatório.**

- `GET /dashboard/administrativo/clientes` roda **uma única consulta** sobre `tbcliente` com LEFT JOIN em `tbcontrato` (soma de `QTDE_LICENCA`) e em `tbcliente_grupo` (nome do grupo).
- O LEFT JOIN é **obrigatório**: cliente sem contrato precisa continuar na lista exibindo `0` licença. Trocar por JOIN interno apagaria da tela os clientes sem contrato.
- Filtros: `busca` + `campo` (`todos`\|`cliente`\|`cnpj`\|`grupo`), `situacao` (`ativo`\|`inativo`) e `grupoId` são aplicados **no banco**; a `ordenacao` (`cliente`\|`licencas-desc`\|`licencas-asc`\|`id`) é aplicada em memória após a projeção, porque depende do somatório.
- Filtro de situação em `tbcliente`: `ativo` cobre `ATIVO` nulo, vazio ou `'S'`; `inativo` cobre qualquer outro valor preenchido.
- `GET /dashboard/administrativo/grupos` faz **duas consultas de propósito**: somar licenças direto na projeção de grupos geraria `SUM` dentro de `SUM`, que o SQL Server recusa (erro 130). Os totais são unidos em memória por `GRUPO_ID`.

