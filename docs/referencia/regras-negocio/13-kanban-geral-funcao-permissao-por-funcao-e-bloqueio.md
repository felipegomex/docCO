---
id: regras-negocio.13
titulo: "Kanban Geral — função, permissão por função e bloqueio"
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
tokens_estimados: 2615
ficha: docs/modulos/kanban.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## Kanban Geral — função, permissão por função e bloqueio

**Situação: implementada.** Fonte da verdade da permissão: [`FuncaoAuthorizationService`](../../../backend/Central_BackEnd/Services/Implantacao/FuncaoAuthorizationService.cs).

### A função é o eixo do quadro

- O eixo é **`tbfuncao`**, a tabela legada unificada de cargo, lida por `tboperador.FUNCAO_ID`. As colunas do eixo renomeadas para `TRF_FuncaoId` (tarefa) e `PRJ_FuncaoId` (projeto) apontam para `FUNCAO_ID`; a `CLK_FuncaoId` (coluna) sobrou do eixo Função, mas **no modelo vigente ela é sempre `NULL`** — a coluna é do catálogo compartilhado (ver a seção "Colunas — catálogo compartilhado e visão por setor").
- **Não existe mais eixo Setor.** `tbsetor`, `tbetapsetor` e as colunas `tboperador.SetorId`, `tbtarefa.TRF_SetorId`, `tbprojeto.PRJ_SetorId` e `tbcolunakanban.CLK_SetorId` foram removidos; o catálogo de etapas padrão virou **`tbfuncaoetapa`** (prefixo `FUN_`, FK `Cascade` para `tbfuncao`).
- **Motivo (decisão de negócio):** nenhum operador tinha `SetorId` e a regra antiga só permitia escrita ao dono do setor, o que na prática restringia o quadro a A/S/F. Como todos os operadores têm `FUNCAO_ID`, "edita o card da própria função" destrava o quadro.
- `tbfuncao` **não ganhou colunas**: não há cor nem ordem nessa tabela. A **cor é derivada de `CLASSIFICACAO` no frontend** ([`funcao-visual.config.ts`](../../../frontend/src/app/shared/config/funcao-visual.config.ts): `Implantador` → `#f59e0b`, `Atendimento` → `#0ea5e9`, `Desenvolvimento` → `#8b5cf6`, `Financeiro` → `#10b981`, resto/`NULL` → `#94a3b8`) e a **ordem dos pills é a ordem de `FUNCAO_ID`** — o frontend não reordena.
- Base real (2026-10-21): `tbfuncao` tem **5** linhas — 1 `Analista de Sistemas` (classificação `Implantador`), 2 `Suporte` (`Atendimento`), 3 `Programador` (`Desenvolvimento`), 4 `Executivo(a) Comercial` e 5 `CIAA`. `FUNCAO_ID` **não é IDENTITY** (o próximo livre é manual) e `DESCRICAO` é `varchar(20)` na base, embora o modelo declare 100.
- **RH e Comercial não existem como função.** Eram linhas de `tbsetor` sem nenhum operador e viraram `FUNCAO_ID NULL` nos cards. Cards nessa situação **não aparecem em nenhum recorte por função** — e desde 2026-10-01, com a pill "Todas" **removida** do Kanban Geral, não aparecem em **lugar nenhum** do quadro: para voltarem, o card precisa receber uma função (linha em `tbfuncao` + ajuste do `FUNCAO_ID`).
- `GET /implantacao/funcoes` é **somente leitura** (1 operação): quem mantém uma função é o cadastro de operadores. Um CRUD ali criaria um segundo cadastro da mesma unidade.
- Mapa usado no backfill da migração: setor 1 `Suporte` → função 2; setor 2 `Implantacao` → função 1; setor 3 `Financeiro` → função 4; setores 4 `RH` e 5 `Comercial` → `NULL`.

### Colunas — catálogo compartilhado e visão por setor (2026-10-20)

**Situação: implementada** em 2026-10-20, com o script `ColunasCompartilhadas_Idempotente.sql` aplicado em `dbBUSINESS_HML` **antes** do deploy. Substitui o modelo de 2026-10-02 ("colunas próprias por função"), que dava a cada setor uma **cópia física** das 7 colunas (35 linhas) e permitia **excluir** a cópia — a comparação entre setores dependia de coincidência, não de regra. Decisões que continuam valendo: **Q3** escrita do catálogo restrita a `Roles = "Administrador"`; **Q4** bloquear desativação de coluna com cards.

- **Catálogo compartilhado.** A coluna é do **catálogo** (`tbcolunakanban`, `CLK_FuncaoId` **nulo**): as mesmas colunas para todos os setores, porque a análise comparativa entre funções exige o mesmo conjunto. `GET /colunas-kanban?todas=true` devolve o catálogo inteiro (tela de administração); sem `todas` devolve o catálogo ativo; com `funcaoId=N` devolve a **visão do setor**.
- **Visão por setor (`tbcolunafuncao`):** `CLF_Visivel` oculta/ativa a coluna no quadro daquele setor e `CLF_NomeAlternativo` é a **máscara de nome** (como a coluna aparece para o setor, **sem** renomear a do catálogo). Sem linha de visão, a coluna vale (padrão do setor). A visão é do **gestor da função** ou do administrador (`GET`/`PUT /colunas-kanban/visao/{funcaoId}`; `403` para os demais).
- **Visibilidade da coluna (substitui a "posse" da spec 3.4):** ao criar/atualizar/mover, a coluna precisa estar **visível no quadro do setor**. Tarefa herda a função do projeto ou, sem projeto, do **criador**; projeto herda a do **criador**. Erro: `400 "Coluna inválida para a função da tarefa/projeto."`; no movimento pelo quadro, `"A coluna de destino não está visível neste quadro."`.
- **Reordenação global (substitui a spec 3.5):** `POST /colunas-kanban/reordenar` reordena o **catálogo inteiro** — uma ordem só, para todos os setores.
- **Coluna não é excluída:** `DELETE /colunas-kanban/{id}` **não existe mais**. O caminho de saída é **desativar** (`CLK_Ativa`), e ele é **global** — a coluna sai do catálogo inteiro, não só de um setor. Desativar coluna **com cards** devolve `400` (`"Não é possível desativar: existem cards nesta coluna."`) — o `FK SET NULL` tiraria o card do quadro em silêncio (Q4).
- **Administração (Q3 / spec 3.8):** escrita do **catálogo** (`POST`/`PUT`/`reordenar`) exige `Roles = "Administrador"`; a **visão do setor** é do gestor da função ou do administrador. A tela `/kanban/admin/colunas` tem **dois blocos**: o **catálogo** (criar, editar, ativar/desativar, reordenar — **sem excluir**) e a **visão por setor** (o componente `app-config-colunas-setor`), com o filtro `?funcaoId=`.
- **Formulário de tarefa/projeto:** o select de colunas lista o **catálogo** — o filtro por função no cliente deixou de ter efeito (a coluna não pertence mais a uma função). Quem decide se a coluna vale é o backend (visibilidade) e quem a oculta é o gestor, na tela de visão do setor.

### Matriz de permissão (`PermissaoFuncao`)

A matriz foi reancorada no **cargo** (`GESTOR_MASTER`) e em `tbgestor`; o eixo `PERFIL_ID`/`SE_ADMIN` sobrevive apenas como ponte para o Master.

| Situação | `PodeVerFuncao` | `PodeEditarFuncao` | `PodeCustomizarFuncao` |
|---|:--:|:--:|:--:|
| `GESTOR_MASTER` (ou ponte `SE_ADMIN = 1`/`PERFIL_ID = 'A'`) | ✅ qualquer função | ✅ qualquer função | ✅ qualquer função |
| Gestor da função (`tbgestor` ativo) | ✅ a própria e as geridas | ✅ a própria e as geridas | ✅ as geridas |
| Operador comum com função | ✅ só a própria | ✅ só a própria | ❌ |
| Operador sem função | ❌ | ❌ | ❌ |
| Card de origem **sem função** (`NULL`) | só Master | só Master | só Master |

- **A leitura deixou de ser aberta.** `PodeVerFuncao` só é `true` para o Master, para a função do próprio operador e para as funções que ele gerencia em `tbgestor`; qualquer outro setor devolve `403` no dashboard, nos projetos e na listagem de tarefas.
- `EhAdmin` = cargo `GESTOR_MASTER` **ou** a ponte legada `SE_ADMIN`/`PERFIL_ID = 'A'`. `EhSuporte` e `EhPerfilF` existem só por compatibilidade e devolvem `false`: `PERFIL_ID 'S'`/`'F'` **não** são mais superusuários do Kanban.
- `FuncoesDeLeitura` = função do operador + funções geridas; é o recorte usado por `ProjetoService.ListarAsync`. Para o Master a coleção é vazia e não há recorte.
- **"Implantador" é `CLASSIFICACAO`, não id.** `FUNCAO_ID == 1` como sinônimo foi substituído por comparação de `Classificacao == "Implantador"` em `AuthService` e em `TarefaService.PodeAtribuirTerceiroAsync` — mais robusto a reordenação/inserção de ids.
- No frontend, `KanbanComponent.podeEditarCard` reproduz a matriz com `ehAdmin`, `cargoCodigo = GESTOR_MASTER`, `funcaoId` e `funcoesGeridas`; card sem função nunca é editável por operador comum, e o drag fica desabilitado.

### Bloqueio como atributo

- A coluna 22 "BLOQUEADO" foi **desativada** e não aparece no quadro (o frontend ainda filtra `id !== 22` e nomes com `BLOQUEAD` por segurança). Mover card não abre nem fecha bloqueio.
- O bloqueio é uma linha de `tbbloqueio` ligada à origem (`BLQ_OrigemTipo` 0 = Tarefa, 1 = Projeto, 2 = Etapa de projeto; `BLQ_OrigemId`), com **motivo obrigatório** (`Motivo do bloqueio é obrigatório.`). Índice único filtrado garante **no máximo um bloqueio aberto por origem**.
- **Dias parado** vem de `BLQ_DataInclusao` da linha aberta (`DateTime.Now - DataInclusao`); não existe campo "desde quando" dedicado. Depois de fechado, `DiasParado` vira o intervalo fechado.
- Reabrir um item **já bloqueado não cria uma segunda linha**: `BloqueioService.AbrirAsync` **atualiza o motivo** da linha aberta. O histórico vem do ciclo fechar → abrir de novo (cada ciclo é uma linha resolvida + uma linha aberta). A reabertura também substitui a marcação antiga, portanto **perde-se o motivo anterior** desse mesmo ciclo.
- Abrir e fechar bloqueio é escrita dentro de uma função e passa pela mesma regra de mover o card (`403 Sem permissão para alterar itens de outra função.`).

### Etapas da função são fixas

- O catálogo de etapas padrão por função vive em `tbfuncaoetapa` (FK `Cascade` de etapa para função). A tarefa guarda os dois ponteiros: `TRF_EtapaTemplateId` (padrão da função) e `TRF_ProjetoEtapaId` (posição real do projeto), e `TarefaService` recusa template de função diferente da função dona do card.
- **Decisão de negócio: as etapas são FIXAS.** O CRUD de `implantacao/etapas-funcao` existe no backend, **mas não é exposto no frontend** — a tela de Administração do quadro apenas lista as etapas e informa que não são editáveis.

### Filtro `?perfil=` do quadro geral

- `GET /implantacao/kanban/board` **não** filtra por perfil, e **projetos não têm filtro por perfil**. O parâmetro `?perfil=` virou **deep link** em 2026-10-01 e, desde 2026-10-20, **nenhum item de menu o envia** (o "Kanban ADM" saiu do menu por duplicar `/kanban`); quando presente na URL, o frontend cruza `board?tipo=tarefas` com `GET /implantacao/tarefas?perfilId=&perfilModo=incluir&funcaoId=&limite=2000` **por id** (`carregarBoardPorPerfil`).
- Consequência intencional: com `?perfil=` ativo, o toggle **Projetos fica vazio** e a tela avisa que o quadro não filtra projetos por perfil. `perfilModo` = `incluir` quando há `?perfil=` na URL e `excluir` nas rotas legadas. O recorte por perfil é sempre o do modo legado (tarefas cujo criador pertence ao perfil).

