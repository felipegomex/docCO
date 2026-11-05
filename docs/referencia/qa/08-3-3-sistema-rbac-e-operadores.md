---
id: sec.08-3-3-sistema-rbac-e-operadores
titulo: "3.3 Sistema — RBAC e operadores"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2970
ficha: docs/modulos/auth-rbac-sistema.md
---
> Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 3.3 Sistema — RBAC e operadores

### RB-01 — Entrar em Funções Administrativas e enxergar o menu

- **Objetivo:** Confirmar que a seção **Sistema** aparece para quem tem leitura em `aba.sistema.funcoes-administrativas` e que a tela abre com os atalhos administrativos.
- **Pré-condições:** operador autenticado; `CARGO_ID` com permissão `Ler` no recurso `aba.sistema.funcoes-administrativas` (Master tem bypass); `PermissoesService` ainda não carregado.
- **Passos:** 1. Entrar com o operador. 2. Abrir o menu **Sistema → Funções Administrativas**. 3. Conferir os cards **Operadores**, **Gestores**, **Permissões** (quando houver `Ler` em `aba.sistema.permissoes`), **Cadastros**, **Colunas do Kanban** e **Auditoria de Logs**. 4. Repetir com um operador **sem** a permissão (ou sem cargo).
- **Resultado esperado:** o menu consulta `GET /api/v1/permissoes/minhas`, esconde o item quando não há `Ler` no recurso e o `permissaoGuard('aba.sistema.funcoes-administrativas')` leva a `/sem-acesso?recurso=aba.sistema.funcoes-administrativas` se a rota for digitada à mão. Com permissão, a tela abre; o card **Permissões** é exibido por RBAC (`recurso: 'aba.sistema.permissoes'`, não por `hasRole('Administrador')`), enquanto Cadastros e Colunas seguem condicionais a `Administrador`. **O backend revalida**: esconder o item não é a barreira.
- **Estado atual/bloqueio:** o guard e o menu são **espelho** do conjunto devolvido pela API; se o RBAC mudar no banco, a tela só reflete no próximo carregamento. As telas de operadores e de funções usam `permissaoGuard` por recurso, mas não `adminGuard`/guard de papel; a autorização efetiva é a policy do backend.
- **Rotas/endpoints/tabelas:** `/sistema/funcoes-administrativas`; `GET /api/v1/permissoes/minhas`; `tbpermissao_recurso`, `tbperm_cargo`, `tbperm_operador`, `tbcargo`, `tboperador`.
- **Status/automação:** **cenário documentado**; não há teste de tela.

### RB-02 — Consultar as próprias permissões e o guard de rota

- **Objetivo:** Verificar que `/sistema/operadores` só abre com `Ler` em `aba.sistema.operadores` e que o conjunto efetivo reflete cargo + exceções.
- **Pré-condições:** operador autenticado; um perfil com permissão e outro sem; opcionalmente uma exceção individual (positiva ou negativa) em `tbperm_operador`.
- **Passos:** 1. Abrir `/sistema/operadores` com o perfil autorizado. 2. Conferir a lista e o seletor de cargos. 3. Repetir com o perfil sem permissão. 4. Criar uma exceção **negativa** para `Ler` e recarregar. 5. Criar uma exceção **positiva** para um operador sem cargo e recarregar.
- **Resultado esperado:** `GET /api/v1/permissoes/minhas` devolve `operadorId`, `cargoId`, `cargoCodigo`, `funcaoId` e `permissoes[]` com `acoes` em bitset; o `permissaoGuard` libera apenas quando `acoes & Ler != 0`; sem a ação, a navegação vai para **`/sem-acesso?recurso=<recurso>`** (com a mensagem e o recurso na tela), não mais para `/`. A **negativa individual vence** o cargo: a ação `Ler` sai do bitset (o recurso pode continuar na lista com outras ações); a positiva concede antes do cargo. Sem cargo e sem exceção, o recurso não entra na lista e a rota cai em `/sem-acesso`. O backend responde **403** se a escrita for tentada sem a policy `perm:aba.sistema.operadores:Editar`.
- **Estado atual/bloqueio:** o frontend não valida assinatura nem recalcula a matriz: confia no JSON de `/permissoes/minhas`; o backend é a barreira final. `GET /api/v1/operadores` e `GET /api/v1/cargos` não têm policy `perm:*` — qualquer autenticado lê.
- **Rotas/endpoints/tabelas:** `/sistema/operadores`; `GET /api/v1/permissoes/minhas`, `GET /api/v1/operadores`, `GET /api/v1/cargos`; `tbcargo`, `tbperm_cargo`, `tbperm_operador`, `tbpermissao_recurso`.
- **Status/automação:** **cenário documentado**; não há teste de tela nem de contrato do `PermissoesService` no frontend.

### RB-03 — Master cria operador e define cargo/setor

- **Objetivo:** Criar um operador pela tela, com cargo e setor, e conferir a ligação com o Kanban legado quando o cargo é Master.
- **Pré-condições:** chamador `GESTOR_MASTER`; `GET /cargos` devolvendo os 11 cargos ativos; `tbfuncao` com setores ativos.
- **Passos:** 1. Abrir `/sistema/operadores` e clicar em **Novo operador**. 2. Informar `OPERADOR_ID` (≤ 15), nome, e-mail, senha (≤ 10), **cargo** e setor. 3. Salvar. 4. Repetir com um `OPERADOR_ID` já existente, um e-mail já usado e uma senha com 11 caracteres. 5. Criar um operador com o cargo `GESTOR_MASTER`.
- **Resultado esperado:** `POST /api/v1/operadores` devolve `201 OperadorDetalhe`; a lista recarrega e o operador aparece com a descrição do cargo ou o código. Duplicidades devolvem `409` (`OPERADOR_DUPLICADO`/`EMAIL_DUPLICADO`); senha com 11 caracteres devolve `400`; cargo/setor inexistente ou inativo devolve `400`. O operador Master nasce com `SE_ADMIN = true` e `PERFIL_ID = 'A'` e a auditoria grava `CREATE` com `OPERADOR_ID=…` (sem senha).
- **Estado atual/bloqueio:** não há `DELETE` — criar errado exige inativar. O e-mail é validado apenas por presença/posição de `@` e unicidade sem caixa, não por formato completo. O `OPERADOR_ID` é imutável na edição.
- **Rotas/endpoints/tabelas:** `/sistema/operadores`; `GET /api/v1/cargos`, `GET /api/v1/operadores`, `POST /api/v1/operadores`; `tboperador`, `tbcargo`, `tbfuncao`, `tbauditoriaimplantacao`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do `OperadorAdminService` no inventário atual.

### RB-04 — Tech Lead restrito ao próprio setor e aos cargos abaixo

- **Objetivo:** Confirmar que o Tech Lead administra só o próprio setor e não cria/altera Master nem outro Tech Lead.
- **Pré-condições:** chamador com cargo `TECH_LEAD_IMPLANTACAO`, `TECH_LEAD_DESENVOLVIMENTO` ou `TECH_LEAD_CIAA`; permissões `aba.sistema.operadores` (`Ler`/`Criar`/`Editar`, escopo `Funcao`); um operador do mesmo setor e outro de setor diferente.
- **Passos:** 1. Editar um operador **do mesmo setor** com cargo abaixo do Tech Lead. 2. Tentar editar um operador de **outro setor**. 3. Tentar criar/alterar um `GESTOR_MASTER` ou outro Tech Lead. 4. Tentar alterar o próprio cargo. 5. Tentar inativar a si mesmo.
- **Resultado esperado:** o passo 1 salva e audita. Os passos 2–5 devolvem **403** com mensagem específica: `"Tech Lead so administra operadores do proprio setor."`, `"Tech Lead nao altera Master nem outro Tech Lead."`, `"Nao e permitido alterar o proprio cargo."` e `"Nao e permitido inativar o proprio operador."`. O Tech Lead só enxerga a tela se tiver a permissão semeada pelo `Permissoes_SeedOperadores_Idempotente.sql` (o seed concede `Ler|Criar|Editar` com escopo `Funcao` aos três cargos).
- **Estado atual/bloqueio:** na **administração de operadores**, a hierarquia continua de aplicação (`Nivel(codigo)` em `OperadorAdminService`), não uma coluna do banco: um cargo novo fora da lista cai em nível 0 e não pode ser administrado por Tech Lead; `TECH_LEAD_ADM` não está nessa lista. Na **matriz de permissões**, a hierarquia vem de `CARGO_Nivel`/`CARGO_FuncaoId` (ver RB-06). O Tech Lead **não** edita operador sem setor (`FuncaoId` nulo) porque a comparação exige o mesmo `FuncaoId`.
- **Rotas/endpoints/tabelas:** `/sistema/operadores`; `PUT /api/v1/operadores/{id}`, `PATCH /api/v1/operadores/{id}/status`, `POST /api/v1/operadores`; `tboperador`, `tbcargo`, `tbperm_cargo`, `tbauditoriaimplantacao`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do escopo Master/Tech Lead no inventário atual.

### RB-05 — Inativar, trocar senha e recusar exclusão

- **Objetivo:** Confirmar que a saída do operador é **inativação**, que a senha respeita o limite de 10 e que o 403 de policy é auditado.
- **Pré-condições:** Master ou Tech Lead com escopo no operador alvo; operador ativo.
- **Passos:** 1. Em `/sistema/operadores`, alternar o status para **inativo** e depois reativar. 2. Trocar a senha com 10 caracteres. 3. Tentar trocar com 11 caracteres. 4. Verificar que **não existe** botão/ação de excluir. 5. Provocar um `403` (por exemplo, Tech Lead tentando editar Master) e conferir a trilha.
- **Resultado esperado:** `PATCH /{id}/status` grava `SE_ATIVO = 'S'/'N'` e audita `ATIVAR`/`INATIVAR`; `PATCH /{id}/senha` com 10 caracteres devolve `200` e audita `SENHA` **sem o valor**; com 11 devolve `400 "A senha deve ter no maximo 10 caracteres (limite da TBOPERADOR)."`. Não há `DELETE` no controller nem na tela. O `403` de uma policy `perm:*` gera em `tbauditoriaimplantacao` uma linha `Entidade = "Autorizacao"`, `Acao = "ACESSO_NEGADO"` com recurso, ação, IP, cargo e função.
- **Estado atual/bloqueio:** a senha continua em `varchar(10)` e é comparada em texto pelo login — o limite é da coluna, não uma política de senha forte. A inativação **não** apaga `tbrefreshtoken`/permissões; o operador inativo é recusado pelo motor (`SE_ATIVO = "N"`).
- **Rotas/endpoints/tabelas:** `/sistema/operadores`; `PATCH /api/v1/operadores/{id}/status`, `PATCH /api/v1/operadores/{id}/senha`, `GET /api/v1/auditoria/implantacao`; `tboperador`, `tbauditoriaimplantacao`, `tbrefreshtoken`.
- **Status/automação:** **cenário documentado**; não há teste automatizado da tela.

### RB-06 — Matriz de permissões: Master, Tech Lead e negação

- **Objetivo:** Confirmar que `/sistema/permissoes` lista só os cargos que o operador gerencia e que o Tech Lead não alcança outro setor, outro Tech Lead nem o Master.
- **Pré-condições:** `Permissoes_Idempotente.sql` aplicado (`CARGO_Nivel`/`CARGO_FuncaoId` preenchidos); um operador `GESTOR_MASTER`, um `TECH_LEAD_CIAA` (`CARGO_FuncaoId = 5`, nível 3), um `CIAA` comum (nível 2, função 5), um cargo de outro setor e um `TECH_LEAD_IMPLANTACAO`; `aba.sistema.permissoes` concedido por `Permissoes_Matriz_Idempotente.sql` aos Tech Leads.
- **Passos:** 1. Entrar como Master e abrir `/sistema/permissoes`; conferir a lista de cargos. 2. Selecionar um cargo comum, alterar ações/escopo e salvar. 3. Entrar como `TECH_LEAD_CIAA` e repetir. 4. Tentar abrir/salvar a matriz de um cargo de outro setor, do outro Tech Lead e do `GESTOR_MASTER` (inclusive pela URL). 5. Entrar como `CIAA` comum e tentar abrir a tela. 6. Depois de salvar a matriz de um cargo do CIAA, conferir as linhas de `tbperm_cargo` com `PEC_FuncaoId = 5`.
- **Resultado esperado:** o Master vê todos os cargos **ativos, exceto `GESTOR_MASTER`**, e salva qualquer um deles; o Tech Lead vê somente o cargo do **próprio setor e de nível inferior** (no exemplo, `CIAA`); os demais cargos devolvem **403** em `GET/PUT /permissoes/matriz/{cargoId}`; o comum não vê o card e a API também nega. `GET /permissoes/cargos-gerenciaveis` devolve a mesma lista da tela. O salvamento desativa apenas `PEC_FuncaoId IS NULL`: a linha por função de `aba.ciaa.ideias` (`PEC_FuncaoId = 5`) continua ativa.
- **Estado atual/bloqueio:** a tela não replica a hierarquia no cliente — ela renderiza o que `cargos-gerenciaveis` devolve; a barreira é `PodeGerenciarCargoAsync` no `GET` e no `PUT`. A mensagem de 403 na tela é genérica (`Você não pode editar a matriz deste cargo.`).
- **Rotas/endpoints/tabelas:** `/sistema/permissoes`; `GET /api/v1/permissoes/cargos-gerenciaveis`, `GET/PUT /api/v1/permissoes/matriz/{cargoId}`; `tbcargo` (`CARGO_Nivel`, `CARGO_FuncaoId`), `tbperm_cargo`, `tbpermissao_recurso`, `tbauditoriaimplantacao`.
- **Status/automação:** **validado em teste** no backend por `PermissoesCargoHierarquiaTests` (**8** casos: Master, Tech Lead e comum; listagem e preservação da linha por função); **não há teste de tela**.

