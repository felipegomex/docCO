---
id: regras-negocio.24
titulo: "Contagens e fatos confirmados"
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
tokens_estimados: 1280
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Contagens e fatos confirmados

- A fórmula de atraso de tarefa usa `DataEntrega ?? DataPrevisao`; a comparação de atraso de projeto exige `Projeto.DataPrevisao` anterior a `DateTime.Today` e exclui `Concluido`/`Cancelado`. **As duas regras e a de etapa estão hoje na mesma fonte** (`RegrasPrazo`), e a decisão é exposta no contrato (`atrasado` no resumo de projeto, no resumo de tarefa e no card do quadro) — o **cliente consome** a decisão do backend; só o helper `ehAtrasada` recalcula, e apenas como **reserva** (quando o endpoint não traz o campo).
- O refresh é rotacionado, revogado e substituído; o cookie chama-se `cc_refresh`, o campo JSON permanece presente como `refreshToken: ""` e o token de acesso fica em memória no frontend.
- A criação/edição de tarefa verifica o responsável principal; a lista `ResponsavelIds` não tem uma autorização uniforme entre os dois fluxos.
- O rótulo de perfil `F` no token passou a `Financeiro`; o `hasRole` do frontend compara também a letra, então o item de menu `roles: ['F']` sobrevive ao rename. A letra que o token carrega (`perfil_id`) é **normalizada** (trim + maiúsculas) e serve à ponte legada de admin; a permissão por função do Kanban vem do cargo `GESTOR_MASTER` + `tbgestor`, não da letra.
- A agenda valida conflito do responsável e retorna `409`; não valida conflitos de participantes, não filtra `Visibilidade` e não restringe propriedade nas leituras.
- A sincronização tarefa–agenda é chamada na mudança de coluna e não é bidirecional.
- O código de projeto é global, começa por `PRJ-` e termina com quatro dígitos; não há sequência transacional.
- A etapa do projeto é obrigatória na tarefa com projeto e precisa pertencer a ele; etapa sem projeto é rejeitada na criação e zera na atualização de tarefa sem projeto; o `PUT` de tarefa não altera o projeto. Essa regra é executada pelo serviço e **não tem suíte de testes no backend**: a cobertura automatizada existente em [`tarefa-form.component.spec.ts`](../../../frontend/src/app/features/implantacao/pages/tarefas/tarefa-form.component.spec.ts) exercita somente o gating e o payload do cliente, não a validação do serviço.
- Faturamento e atraso derivados de `tbcontrato` estão **desligados por decisão de produto** (`FlagsDashboard.FaturamentoLiberado = false`): a API devolve `0` e lista de atrasos vazia em vez de publicar a projeção `QTDE_LICENCA × VALOR_UNITARIO`. Reajustes (datas) e custos/orçamentos (`tbcusto`/`tborcamento`, tabelas que o sistema gerencia) **não** são afetados. Para religar basta trocar a constante para `true`.
- Contrato sem `DATA_PROXIMO_REAJUSTE` devolve `diasParaReajuste = null`; antes ele virava `0` e entrava como falso positivo no filtro "reajuste em até 30 dias" (101 falsos na base de homolog, contra 1 real).
- **RBAC:** precedência confirmada em `PermissaoService` (negativa individual > positiva individual > cargo > negado), escopo `Proprio`/`Funcao`/`Tudo`, bypass de `GESTOR_MASTER` e policies `perm:<recurso>:<acao>`; os 403 de policy são auditados em `tbauditoriaimplantacao` (`ACESSO_NEGADO`). Em `dbBUSINESS_HML` (2026-10-21): 10 cargos, 8 `GESTOR_MASTER`, 1 `TECH_LEAD_IMPLANTACAO`, 32 sem cargo; `tbfuncao` 4 = `Executivo(a) Comercial` e 5 = `CIAA`.
- Nenhuma regra de UI foi inferida; as correções listadas são apenas recomendações.
- **Acessos Empresas:** soft delete em cascata no cliente; flags `CAO_Tem*` derivadas e recalculadas pelo serviço; grupos somente leitura via `tbcliente_grupo` (sem CRUD, sem `409`); `POST /clientes` com `clienteId` cria unidade a partir do ERP sem duplicar (`jaExistia`); o mesmo `POST` aceita o agregado **`acessos`** (lista por tipo) que grava empresa + acessos numa **transação única**, com validação por item (servidor/RDP min. 3, usuário min. 3, senha min. 6, porta 1–65535) e auditoria de `CREATE` por registro; `POST /rdp` exige IP ou Servidor; portas 1–65535 com defaults 3389/1433/1010; `TipoVpn` e `Ambiente` validados por lista de constantes.
- **Acessos Empresas — armadilha ativa:** `SenhaCifrada`/`TemSenha` são deliberadamente declaradas em cada classe concreta, **fora** de `AcessoBase` (sombreamento de propriedade). Regressão coberta por `backend/Central_BackEnd.Tests/TemSenhaTests.cs`.
- **Acessos Empresas — cobertura de teste:** o backend tem suíte própria para a matriz de permissão por função, o bloqueio e o quadro (`KanbanFuncaoTests`). A contagem por arquivo está em [`backend.md`](../backend.md) § 5.5, que inclui agora `RegrasPrazoTests` (regra única de prazo) e `PerfilOperadorTests` (leitura do `PERFIL_ID`). O gate `scripts/validate.ps1` executa `dotnet test` no passo **5/7** (suíte de `backend/Central_BackEnd.Tests`) e `ng test` no passo **6/7** (headless); se ele ficar vermelho, o commit/deploy não deve seguir. O frontend tem `karma.conf.js` e o pipeline executa a suíte.


