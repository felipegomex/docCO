---
id: 05-4-roteamento.05
titulo: "4.4.1 Kanban Geral (rotas de topo)"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 2618
ficha: docs/modulos/kanban.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 4.4.1 Kanban Geral (rotas de topo)

Declaradas em [`features.routes.ts`](../../../../frontend/src/app/features.routes.ts), fora de `implantacao.routes.ts`, e carregadas com `loadComponent`. Herdam o `authGuard` do shell em `/`.

| URL | Componente | Dados/observações |
|---|---|---|
| `/kanban` | `KanbanComponent` | `data: { modo: 'geral' }` — é o que liga o quadro por função. Aceita `?funcaoId=`, `?tipo=` e `?perfil=`. |
| `/kanban/gargalos` | `GargalosComponent` | Cards bloqueados com dias parado; aceita `?funcaoId=`. |
| `/kanban/configurar` | `ConfigColunasSetorComponent` | **Visão do setor** sobre o catálogo de colunas (visível + máscara de nome). Rota com `data: { configurarSetor: true }`; aceita `?funcaoId=` e, na falta, usa a função do usuário logado. |
| `/kanban/admin/colunas` | `AdminColunasComponent` | **Catálogo de colunas** compartilhado + visão por setor + lista somente-leitura de funções + leitura das etapas da função; aceita `?funcaoId=` (filtro da visão e das etapas). |

`/kanban` é o **único caminho do quadro**: cada setor (Suporte, Implantação, Desenvolvimento, CIAA) tem o seu item **Kanban** apontando para cá com `?funcaoId=` (o ADM perdeu o item na sidebar em 2026-10-22). O item antigo "Kanban ADM" — que duplicava a mesma rota — saiu do menu em 2026-10-20 (Fase 0), e com ele o atalho que enviava `?perfil=F`; o parâmetro `?perfil=` continua aceito como **deep link** (ver a subseção abaixo).

#### Modo geral × modo legado no mesmo componente

`KanbanComponent` ([`kanban.component.ts`](../../../../frontend/src/app/features/implantacao/pages/kanban/kanban.component.ts)) tem dois regimes, decididos por `route.data.modo`. **Desde 2026-10-20 (Fase 0) só o modo geral é alcançável** — a tabela abaixo documenta o regime legado como código dormente do componente:

| Aspecto | Modo geral (`modo: 'geral'`) — **único alcançável** | Modo legado (rotas antigas sem `modo`) — **sem rota** |
|---|---|---|
| Fonte de colunas e cards | `GET /implantacao/kanban/board` (`FuncoesService.obterBoard`) | `GET /implantacao/colunas-kanban?todas=true` (scaffold com **todas** as colunas) + `GET /implantacao/tarefas` |
| Tabela de funções | Pills de função, com **default = função do usuário logado** (sem ela, a primeira da lista); **a pill "Todas" foi removida em 2026-10-01** — o quadro sempre abre sob uma função | — |
| Tipo de card | Toggle Tarefas / Projetos / Ambos (`?tipo=`) | Só tarefas |
| Card de projeto | Sim (`tipoCard: 'projeto'`, percentual, projeto de origem) | Não |
| Criação de card na tela | Sim (formulário inline também no modo geral, após remoção de `*ngIf="!modoGeral"`). **Na aba Projetos o gatilho vira "Adicionar Projeto" e navega para `/implantacao/projetos/novo`** (`somenteProjetos()`): o board `tipo=projetos` só devolve cards de `tbprojeto`, então uma tarefa criada ali nasceria fora do filtro e sumiria no próximo `carregar()` (o `POST .../coluna` do drag não toca no vínculo — `TarefaService.MudarColunaAsync` preserva `ProjetoId`) | Sim, formulário inline |
| Coluna "BLOQUEADO" | Filtro explícito (`id !== 22` e nome com `BLOQUEAD`), porque a coluna está inativa | Não se aplica |
| Permissão | `podeEditarCard` por função: `GESTOR_MASTER` ou `ehAdmin` edita tudo; o gestor edita as funções de `funcoesGeridas` (`GET /gestores/minhas-funcoes`); o operador comum edita só a própria função; card sem função nunca é editável pelo comum. Drag e menu desabilitados quando não pode | Edição liberada |

- O modo é resolvido **pela rota**, não pela URL: `/kanban` declara `data.modo = 'geral'` e é a **única rota que carrega o componente** — `/admin/kanban` foi removida e `/implantacao/kanban` virou redirect em 2026-10-20 (Fase 0). Por isso o `ngOnInit` perdeu o fallback `router.url.includes('/admin/kanban')`: o recorte por perfil vem só do deep-link `?perfil=`, e o ramo legado permanece no componente como código dormente.
- pills e toggle gravam `?funcaoId=`/`?tipo=` na URL (deep-link, `replaceUrl`), e a função só é sobrescrita pela função do usuário enquanto nenhum `?funcaoId=` explícito chegar.
- **Badge de função** em cada card (ponto com a cor derivada de `classificacao`) e **card de projeto** com percentual; a coluna de bloqueio virou selo (`t.bloqueada` + `bloqueadaMotivo` + `diasParado`).
- **Cor por `CLASSIFICACAO`, não por id.** [`funcao-visual.config.ts`](../../../../frontend/src/app/shared/config/funcao-visual.config.ts) é a fonte única do mapa (`Implantador` → `#f59e0b`, `Atendimento` → `#0ea5e9`, `Desenvolvimento` → `#8b5cf6`, `Financeiro` → `#10b981`, classificação nula/desconhecida → `#94a3b8`), usado ao mesmo tempo no pill, no badge do card e na lista de gargalos. A ordem dos pills é a **ordem de `FUNCAO_ID`** devolvida pelo backend: o componente não reordena.
- Atalhos no topo: **Gargalos** (com contador de bloqueados do board), **Configurar o quadro do meu setor** (link para `/kanban/configurar?funcaoId=`, visível para o **gestor da função selecionada** ou o administrador — espelha `ColunasKanbanController.PodeConfigurarSetorAsync`) e **Administração do quadro** (visível para quem pode gerenciar).
- **Máscara de nome:** o quadro exibe o nome **efetivo** da coluna (com a máscara do setor) e, quando há máscara, a dica `Catálogo: {nomeCatalogo}` — a comparação entre setores é sempre pelo nome do catálogo.
- **Filtro `?perfil=`**: o board não filtra por perfil, então `carregarBoardPorPerfil` cruza `board?tipo=tarefas` com `GET /implantacao/tarefas?perfilId=&perfilModo=incluir&funcaoId=&limite=2000` **por id**. Projetos **não** têm filtro por perfil e ficam ocultos (a tela avisa). Se a consulta de perfil falhar, o quadro abre **vazio** em vez de mostrar tudo. O parâmetro é **deep link** desde 2026-10-01 e, desde 2026-10-20, **nenhum item de menu o envia** (o "Kanban ADM" saiu).
- **UX read-only**: `podeEditarCard` reproduz `PermissaoFuncao.PodeEditarFuncao` do backend: `GESTOR_MASTER`/`ehAdmin` edita qualquer função; o gestor edita os cards das funções em `funcoesGeridas` (`GET /gestores/minhas-funcoes`); os demais só a própria função; card sem função nunca. O espelho usa `ehAdmin`, `cargoCodigo = GESTOR_MASTER` e `funcaoId`; ações do menu, edição inline, reatribuação e lote ficam desabilitados e o toast de recusa é `Sem permissão para alterar itens de outra função.` O backend revalida (403).
- Aliases de componente preservados: `UsuarioDropdownComponent` importa `KanbanComponent` e `TarefaFormComponent`; a duplicação de tarefa continua vindo do menu do Kanban.

#### Gargalos

[`gargalos.component.ts`](../../../../frontend/src/app/features/implantacao/pages/kanban/gargalos/gargalos.component.ts): lista os bloqueios abertos (tarefa, projeto e etapa) com **chips de dias parado** e severidade (`ok` < 3 dias, `atencao` até 7, `critico` acima), ordenação do mais antigo para o mais recente, alerta na linha a partir de 7 dias, resumo com total/maior/média/críticos, filtro por função em `?funcaoId=` e ação de **fechar bloqueio** (visível apenas para quem pode editar a função — o mesmo `g.funcaoId === u.funcaoId` do quadro).

#### Administração do quadro

[`admin-colunas.component.{ts,html,scss}`](../../../../frontend/src/app/features/implantacao/pages/kanban/admin-colunas/admin-colunas.component.ts), em **quatro blocos**:

1. **Catálogo de colunas (compartilhado)** — o **mesmo conjunto de colunas para todos os setores** (a análise comparativa entre funções exige o mesmo conjunto). A tabela é uma só (não mais agrupada por função): criar (**Nova coluna**), editar nome/cor/**limite WIP** (ação explícita para **limpar** o limite, que o `PUT` ignora quando vem `null`), **ativar/desativar** e **reordenar o catálogo inteiro** (`POST .../reordenar` — uma ordem só, para todos). ⚠️ **Não existe mais excluir coluna**: o caminho de saída é **desativar**, e ele é **global** (a coluna sai do catálogo inteiro); o backend recusa desativar se houver cards na coluna.
2. **Visão por setor** — o bloco `app-config-colunas-setor` (ver abaixo): o que cada função **vê** do catálogo (quais colunas aparecem e a máscara de nome). O setor vem do filtro `?funcaoId=`.
3. **Funções** — lista **somente leitura** com `totalOperadores`. O painel "Setores" virou lista de funções: **sem CRUD, sem cor e sem ordem**. Quem cria e mantém uma função é o cadastro de operadores (`tbfuncao`), não esta tela.
4. **Etapas da função** — **somente leitura**, com o aviso de que são fixas por definição do processo. O CRUD existe no backend, mas não é exposto aqui.

O parâmetro `?funcaoId=` filtra a **visão por setor** e as etapas: vindo do `/kanban`, o link "Administração do quadro" propaga a função selecionada. Sem o parâmetro, a visão usa a função do usuário logado (ou nenhuma).

O acesso é restrito na UI a quem pode gerenciar o quadro (`ehGestor`: `ehAdmin`/`GESTOR_MASTER` ou gestor da função via `funcoesGeridas`); o backend exige `Administrador` nas escritas do **catálogo** e das etapas da função.

#### Visão do setor — `app-config-colunas-setor`

[`config-colunas-setor.component.{ts,html,scss}`](../../../../frontend/src/app/features/implantacao/pages/kanban/config-colunas-setor/config-colunas-setor.component.ts) (selector `app-config-colunas-setor`, standalone). Uma linha por coluna do catálogo com **interruptor de visível** e **campo de máscara de nome**; salvar grava a visão do setor por `PUT /implantacao/colunas-kanban/visao/{funcaoId}` (o serviço `ColunasKanbanService.salvarVisao`).

- **Onde aparece:** embutido na tela de Administração do quadro (bloco 2, com o setor do filtro `?funcaoId=`) e como **página própria** em `/kanban/configurar` (rota com `data: { configurarSetor: true }`), onde o setor vem de `?funcaoId=` ou, na falta, da função do usuário logado; a troca de setor é gravada na URL.
- **Quem edita:** o **gestor da função** ou o **administrador** — o backend responde `403` para os demais, e a tela mostra o aviso de que a visão do setor é editada pelo gestor da função (não um erro genérico). O espelho no cliente usa `funcoesGeridas` (`GET /gestores/minhas-funcoes`) e `ehAdmin`.
- A máscara inicial é o nome efetivo quando ele difere do catálogo; campo vazio = usa o nome do catálogo.

