---
id: 05-4-servicos-principais.17
titulo: "Espelhos legados do módulo"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 640
ficha: docs/modulos/README.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### Espelhos legados do módulo

| Espelho | Ponto de atenção |
|---|---|
| `TbContratoLegado` | `DIA_FATURAMENTO` é o **dia do mês** (`short?`), não uma data: a coluna `DATA_FATURAMENTO` **não existe** e mapear-la derrubava todos os endpoints que leem `tbcontrato`. `QTDE_LICENCA` é `smallint` (`short?`), não `decimal`. Propriedades calculadas `[NotMapped]`: `EhAtivo` (`ATIVO == 'S'` ou `SITUACAO == "Entregue"`) e `ValorMensal` (`(QtdeLicenca ?? 1) * (ValorUnitario ?? 0)`). `EhAtivo` substituiu a condição duplicada em `AdministrativoService` e `FinanceiroService` — a regra antiga testava `SITUACAO == "Ativo"`, valor que nunca ocorre no legado. |
| `ClienteGrupoLegado` | Nova entity para `tbcliente_grupo` (`GRUPO_ID smallint` PK, `NOME`, auditoria), com `ExcludeFromMigrations()`. `ClienteLegado.GrupoId` (`short?`) e `TbContratoLegado.Cliente` são navegações de leitura com `Restrict`. |
| `ClienteLegado` | Ganhou `GrupoId` e a coleção `Contratos`. Única entidade de cliente do modelo; a escrita continua sendo o `INSERT` bruto de `ComercialService.CriarClienteAsync`. |

As constantes de domínio ficam em [`ConstantesDashboard.cs`](../../../../backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs): etapas comerciais (Prospecção, Apresentação, Proposta, Negócio Fechado), status de orçamento (Rascunho, Enviado, Aprovado, Faturado), tipos de custo (Salário, Imposto, Estrutura, Outro), respostas de proposta (SIM, NÃO, AGUARDANDO) e a classe de interruptores **`FlagsDashboard`**.

`FlagsDashboard.FaturamentoLiberado` é `const bool = false` e **desliga de propósito** todo valor de faturamento e atraso derivado de `tbcontrato`: o serviço entrega `0` (ou lista vazia) em vez de publicar a projeção `QTDE_LICENCA × VALOR_UNITARIO`, porque o legado não tem valor total nem data de faturamento e a regra `EhAtivo` inclui `SITUACAO = 'Entregue'` (status de entrega, não de cobrança). A informação ainda não foi liberada pelo Financeiro. **Para religar basta trocar `false` → `true`** — do lado da API não há migration, mudança de DTO nem de rota; apenas os componentes Angular, que esconderam esses campos, precisam ser restaurados. Detalhes e números de referência em [`03-REGRAS-NEGOCIO.md`](../../regras-negocio/INDEX.md).

