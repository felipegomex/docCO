---
id: regras-negocio.11
titulo: "RBAC — cargo, recurso, ação e escopo"
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
tokens_estimados: 1562
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

## RBAC — cargo, recurso, ação e escopo

**Situação: implementado.** Motor único: [`PermissaoService`](../../../backend/Central_BackEnd/Services/Permissoes/PermissaoService.cs); contrato em [`IPermissaoService`](../../../backend/Central_BackEnd/Services/Permissoes/IPermissaoService.cs). A decisão de **linha** (dono do card, gestor da função, setor do registro) continua nos serviços de domínio; o motor responde "pode executar esta ação neste recurso?" e com qual escopo.

**Eixos diferentes.** `tboperador.FUNCAO_ID` é o **setor** (Kanban, dono do card e escopo de leitura). `tboperador.CARGO_ID` é a **responsabilidade** (Suporte N1, Tech Lead, Desenvolvedor, CIAA etc.) e aponta para `tbcargo`. A permissão **não** é lida do nome do cargo: vem de `tbperm_cargo` (cargo) e `tbperm_operador` (exceção individual).

**Precedência (nesta ordem):**

| # | Fonte | Regra |
|---:|---|---|
| 1 | `GESTOR_MASTER` | Bypass integral: `ResolverAsync` devolve `Permitido = true` com escopo `Tudo` para qualquer recurso/ação; `ObterMinhasAsync` devolve **todas** as ações em **todos** os recursos ativos. É decidido pelo `CARGO_Codigo`, não pelo nome/descrição. |
| 2 | Exceção individual **negativa** | `tbperm_operador` com `PEO_Negativa = 1` e a ação no bitset: **nega** e vence a positiva individual e o cargo. O bypass do `GESTOR_MASTER` é resolvido **antes** (item 1). |
| 3 | Exceção individual **positiva** | `tbperm_operador` com `PEO_Negativa = 0` e a ação no bitset: concede antes de consultar o cargo. |
| 4 | Permissão do **cargo** | `tbperm_cargo` com `PEC_Ativo = 1` do `CARGO_ID` do operador, com a ação no bitset. |
| 5 | Sem cargo ou sem linha | `CARGO_ID` nulo (Visualizador) ou nenhuma linha aplicável = **negado**. `NULL` não é "tudo liberado". |

**Escopo (`EscopoPermissao`)** — o par `PEO_FuncaoId`/`PEC_FuncaoId` + `Escopo`:

| Escopo | Valor | Quando permite |
|---|---:|---|
| `Proprio` | 1 | `funcaoAlvo` nulo **ou** igual à função do operador. |
| `Funcao` | 2 | `funcaoAlvo` nulo, igual à função do operador **ou** a uma função que ele gerencia (`IGestorFuncaoService.MinhasFuncoesGeridasAsync`). |
| `Tudo` | 3 | Qualquer alvo, sem recorte de setor. |

A linha de permissão com `FuncaoId = NULL` vale para **todas** as funções; com `FuncaoId` preenchido, só vale quando o alvo é aquela função. O handler de policy chama o motor com `funcaoAlvo: null`; a decisão por alvo é usada pelo código que conhece o registro.

**Hierarquia de cargos e edição da matriz (2026-10-27).** `tbcargo` passou a ter `CARGO_Nivel` e `CARGO_FuncaoId`: `GESTOR_MASTER` nível 4 (global), Tech Lead nível 3 (no setor do cargo), cargos comuns nível 2 e `VISUALIZADOR` nível 1. O setor do cargo vem de `CARGO_FuncaoId` e é **diferente** de `tboperador.FUNCAO_ID`. A regra de quem abre/salva a matriz está em `PodeGerenciarCargoAsync`/`PodeGerenciar`:

| Chamador | Pode editar a matriz de… |
|---|---|
| `GESTOR_MASTER` | **qualquer cargo, exceto o próprio `GESTOR_MASTER`** — o Master não entra na matriz porque tem bypass integral |
| Tech Lead | somente cargo do **mesmo setor** (`CARGO_FuncaoId` igual ao do cargo do operador) e com **nível inferior** ao dele |
| Qualquer outro cargo | nada (`403`) |

`ListarCargosGerenciáveisAsync` aplica a mesma regra em `GET /api/v1/permissoes/cargos-gerenciaveis`, e `GET`/`PUT /api/v1/permissoes/matriz/{cargoId}` usam `PodeGerenciarCargoAsync` — a tela deixou de ser exclusiva do Master. Ao salvar, `SubstituirMatrizAsync` desativa **somente** as linhas gerais (`PEC_FuncaoId IS NULL`); as concessões por setor (`PEC_FuncaoId` preenchido, como CIAA → Ideias) são preservadas.

**Policy e auditoria.** O formato é `perm:<recurso>:<acao>` (ex.: `perm:aba.sistema.operadores:Editar`), traduzido por [`PermissaoPolicyProvider`](../../../backend/Central_BackEnd/Authorization/PermissaoPolicyProvider.cs) em `PermissaoRequirement`; a ação é o **nome** do enum (`Ler`, `Criar`, `Editar`, `Excluir`, `Aprovar`, `Gerir`, `Auditar`) e o recurso é a chave de `tbpermissao_recurso`. O 403 de uma policy `perm:*` é gravado uma única vez por [`AuditoriaNegacaoHandler`](../../../backend/Central_BackEnd/Authorization/AuditoriaNegacaoHandler.cs) em `tbauditoriaimplantacao` como `Entidade = "Autorizacao"`, `Acao = "ACESSO_NEGADO"`, com recurso, ação, IP, cargo e função na observação.

**Conjunto efetivo (`GET /api/v1/permissoes/minhas`).** `ObterMinhasAsync` parte dos recursos ativos, soma (OR) as ações das linhas de cargo aplicáveis, aplica as exceções individuais (positivas somam, negativas subtraem com `& ~`) e mantém o **maior** escopo. Recursos que ficam sem nenhuma ação são omitidos. O resultado é cacheado no escopo da requisição.

**Recursos da reorganização de 2026-10-22.** `Permissoes_Matriz_Idempotente.sql` cadastra os recursos que faltavam à navegação por setor: `aba.ciaa.projetos`, `aba.ciaa.ideias`, `aba.implantacao.ideias`, `aba.suporte.whatsapp.respostas-rapidas`, `aba.ferramentas.mock-api` e `aba.ferramentas.mock-request`. A rota de `aba.ferramentas.base-conhecimento` foi atualizada para `/base-conhecimento`; `aba.ferramentas.mock-*` nascem com `PER_Rota = NULL` (itens "em breve"). A tela `/sistema/permissoes` continua sendo o ponto de controle: os recursos novos aparecem automaticamente depois de aplicar o SQL, sem alteração de código.

**Atualização de 2026-10-27.** O mesmo script passou a conceder `aba.sistema.permissoes` (`Ler`, escopo `Funcao`) aos quatro Tech Leads — a edição continua limitada por setor/nível — e a criar a linha por função de `aba.ciaa.ideias` (`PEC_FuncaoId = 5`, `Ler`, escopo `Funcao`) para **todo cargo não-Master**. Como o motor aplica a linha a quem pertence ao setor, o item fica liberado para **qualquer operador com `FUNCAO_ID = 5`**, independentemente do cargo. A ordem de aplicação é `Permissoes_Idempotente.sql` (colunas `CARGO_Nivel`/`CARGO_FuncaoId` e carga dos níveis) **antes** de `Permissoes_Matriz_Idempotente.sql`.

