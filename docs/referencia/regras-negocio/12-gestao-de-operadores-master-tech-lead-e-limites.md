---
id: regras-negocio.12
titulo: "Gestão de operadores — Master, Tech Lead e limites"
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
tokens_estimados: 866
ficha: docs/modulos/dashboard.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

## Gestão de operadores — Master, Tech Lead e limites

**Situação: implementado.** Serviço: [`OperadorAdminService`](../../../backend/Central_BackEnd/Services/Shared/OperadorAdminService.cs); endpoints em [`OperadoresController`](../../../backend/Central_BackEnd/Controllers/OperadoresController.cs). O recurso das telas de Sistema é `aba.sistema.operadores`.

| Regra | Comportamento confirmado |
|---|---|
| Quem administra | `GESTOR_MASTER` administra **qualquer** operador/cargo/setor. `TECH_LEAD_IMPLANTACAO`, `TECH_LEAD_DESENVOLVIMENTO` e `TECH_LEAD_CIAA` administram **só o próprio setor** (`chamador.FuncaoId == alvo.FuncaoId`) e **cargos de nível abaixo** do Tech Lead. Qualquer outro cargo: `403 "Somente Master e Tech Lead administram operadores."`. |
| Hierarquia de níveis | `GESTOR_MASTER` 100; `TECH_LEAD_*` 80; `SUPORTE_N2` 60; `SUPORTE_N1`/`IMPLANTACAO`/`DESENVOLVEDOR`/`CIAA` 40; `VISUALIZADOR` 10; código desconhecido 0. |
| Tech Lead × Master/Tech Lead | Tech Lead **não cria nem altera** Master ou outro Tech Lead: `403 "Tech Lead nao altera Master nem outro Tech Lead."`. Só o Master altera esses cargos. |
| Autoproteção | Ninguém altera o **próprio cargo** (`403`) e ninguém **inativa a si mesmo** (`403`). |
| Exclusão | **Não existe `DELETE /operadores`.** A saída é **inativar** (`PATCH /{id}/status` com `seAtivo = "N"`); o registro e o histórico permanecem. |
| Senha | Obrigatória, **máximo de 10 caracteres** (`TBOPERADOR.SENHA varchar(10)`); acima disso `400`. Continua no formato atual (comparada em texto pelo login). A auditoria `SENHA` registra só `SenhaAlterada = true`. |
| E-mail | Opcional, `<= 100` caracteres, precisa conter `@` e não pode começar/terminar com ele. A unicidade é **sem diferenciar caixa**; duplicado devolve `409 "E-mail ja cadastrado em outro operador."` (`EMAIL_DUPLICADO`). Não é validação completa de formato. |
| `OPERADOR_ID` | Obrigatório, `<= 15` caracteres; duplicado devolve `409` (`OPERADOR_DUPLICADO`). |
| Cargo/setor | `CargoId` é obrigatório e precisa existir **ativo**; `FuncaoId` é opcional e, se informado, precisa existir ativo. |
| Ponte com o Kanban legado | Criar ou **promover** a `GESTOR_MASTER` grava `SE_ADMIN = true` e `PERFIL_ID = 'A'`. **Rebaixar não limpa** essas duas portas automaticamente. |
| Auditoria | `CREATE`, `UPDATE`, `ATIVAR`, `INATIVAR` e `SENHA` vão para `tbauditoriaimplantacao` com retrato do operador (sem senha) e observação `OPERADOR_ID=…`. |

**Carga inicial no `dbBUSINESS_HML` (2026-10-21).** `Permissoes_CargaInicial_Idempotente.sql` mapeia `SE_ADMIN = 1`/`PERFIL_ID = 'A'` → `GESTOR_MASTER` e gestor ativo das funções 1/3/5 → Tech Lead correspondente, **sem** alterar `SE_ADMIN`/`PERFIL_ID` e só preenchendo `CARGO_ID NULL`. Resultado: **8** `GESTOR_MASTER`, **1** `TECH_LEAD_IMPLANTACAO` e **32** operadores sem cargo (Visualizador). O seed `Permissoes_SeedOperadores_Idempotente.sql` cria os recursos `aba.sistema.funcoes-administrativas` e `aba.sistema.operadores` e concede `Ler|Criar|Editar` (7) com escopo `Funcao` (2) aos três Tech Leads. `tbfuncao` tem 5 linhas, com `FUNCAO_ID` 4 = `Executivo(a) Comercial` e 5 = `CIAA`.

