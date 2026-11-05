---
id: sec.11-6-admin-kanban-f
titulo: "6. Admin/Kanban F"
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
tokens_estimados: 1428
ficha: docs/modulos/kanban.md
---
> Ficha: [Kanban Geral](../../modulos/kanban.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 6. Admin/Kanban F

### AK-01 — Gerenciar Tipos de Projeto

- **Objetivo:** Manter os tipos de projeto disponíveis para o formulário de Projetos.
- **Pré-condições:** usuário com perfil `Administrador`; `adminGuard` aprovado.
- **Passos:** 1. Abrir `/admin/cadastros/tipos-projeto`. 2. Criar em `/admin/cadastros/tipos-projeto/novo`. 3. Editar em `/admin/cadastros/tipos-projeto/editar/:id`. 4. Excluir com confirmação.
- **Resultado esperado:** a listagem mostra código, nome, indicador de cliente obrigatório, ordem e ativo; o código não é editável; criação exige código/nome/ordem; edição permite nome, ordem, cliente obrigatório e ativo; exclusão pode ser recusada pelo backend quando há vínculo.
- **Estado atual/bloqueio:** limitação conhecida: `ClienteObrigatorio` é salvo no tipo, mas a obrigatoriedade não é aplicada uniformemente ao formulário de Projeto nem no `ProjetoService`; validações de cliente e regras de autorização precisam ser centralizadas e testadas.
- **Rotas/endpoints/tabelas:** `/admin/cadastros/tipos-projeto`, `/novo`, `/editar/:id`; `GET /api/v1/implantacao/tipos-projeto?apenasAtivos=true`, `GET /api/v1/implantacao/tipos-projeto/{id}`, `POST /api/v1/implantacao/tipos-projeto`, `PUT /api/v1/implantacao/tipos-projeto/{id}`, `DELETE /api/v1/implantacao/tipos-projeto/{id}`; todas exigem `[Authorize]` e as escrita exigem `Administrador`; `tbtipoprojeto`.
- **Status/automação:** **cenário documentado**; não há teste automatizado do CRUD administrativo.

### AK-02 — Operar o Kanban do perfil F (deep link de perfil)

- **Objetivo:** Conferir o recorte por perfil do quadro geral, que virou **deep link** (`?perfil=`): o item de menu "Kanban ADM" saiu em 2026-10-20 (plano qualidade-logica, fase 0) por duplicar a rota `/kanban` — hoje **nenhum item de menu** envia o parâmetro.
- **Pré-condições:** usuário autenticado cujo `PERFIL_ID` é `F`; a rota `/kanban` disponível.
- **Passos:** 1. Abrir `/kanban` (item **Kanban** da seção do setor) e confirmar a URL **sem** `?perfil=F`. 2. Alternar entre Tarefas, Projetos e Ambos e navegar pelo quadro. 3. (Deep link) abrir `/kanban?perfil=F` digitando na URL e conferir o aviso de filtro por perfil.
- **Resultado esperado:** sem `?perfil=`, o quadro é o Kanban Geral puro (padrão de função do usuário, sem recorte de perfil e sem aviso). Com `?perfil=F` na URL (deep link), `perfilModo` = **incluir**, montado pelo cruzamento `GET /implantacao/kanban/board?tipo=tarefas` com `GET /implantacao/tarefas?perfilId=F&perfilModo=incluir&funcaoId=&limite=2000` por id, e a tela avisa que **o quadro não filtra projetos por perfil**: no toggle *Projetos* o quadro fica vazio de propósito. Sem `?perfil=` o modo volta a `excluir`.
- **Estado atual/bloqueio:** `PERFIL_ID = 'F'` **não** é mais superusuário; a edição segue o cargo `GESTOR_MASTER` + `tbgestor`. O rótulo do perfil agora é `Financeiro`. O recorte `?perfil=` é alcançável **só pela URL digitada** — o menu deixou de oferecer atalho, então o caminho depende de deep link compartilhado. O cruzamento por id do deep link depende do teto de 2000 tarefas e falha para o quadro em branco (com aviso) quando a consulta de perfil não responde.
- **Rotas/endpoints/tabelas:** `/kanban` (e `/kanban?perfil=F` como deep link); `GET /api/v1/implantacao/kanban/board?tipo=tarefas`, `GET /api/v1/implantacao/tarefas?perfilId=F&perfilModo=incluir&limite=2000`; `tbtarefa`, `tbfuncao`, `tbcolunakanban` e `tboperador`.
- **Status/automação:** **cenário documentado**; não há teste do filtro por perfil.

### AK-03 — Conferir as entradas administrativas

- **Objetivo:** Verificar quais telas administrativas são realmente roteadas e o que restou das entradas compartilhadas.
- **Pré-condições:** usuário com perfil `Administrador`; `adminGuard` aprovado.
- **Passos:** 1. Abrir `/admin`. 2. Confirmar que o caminho cai em cadastros. 3. Conferir que não existem mais atalhos administrativos para Kanban, Tarefas e Projetos.
- **Resultado esperado:** `/admin` cai em `/admin/cadastros/tipos-projeto`. As rotas `/admin/kanban`, `/admin/tarefas` e `/admin/projetos` **foram removidas em 2026-10-20** (plano qualidade-logica, fase 0): reaproveitavam os mesmos componentes de `/kanban` e `/implantacao/*`, e nenhum link do app apontava para elas — as telas seguem acessíveis nos caminhos canônicos. O componente e o serviço de dashboard administrativo do frontend também saíram (não tinham rota); o endpoint do backend continua registrado.
- **Estado atual/bloqueio:** limitação conhecida: o serviço/controller do Admin Dashboard **do backend** existem e são registrados no DI, mas não há rota nem menu que os consuma; se a API for chamada diretamente, os totais globais não usam todos o mesmo filtro de arquivamento e os contadores por função são retornados como zero, tornando as métricas potencialmente inconsistentes. O `AdminDashboardComponent` do frontend foi removido nesta fase; o endpoint `/api/v1/admin/dashboard` permanece sem tela.
- **Rotas/endpoints/tabelas:** `/admin`, `/admin/cadastros/tipos-projeto`; `adminGuard` + `authGuard`; endpoints e tabelas de cadastro são os mesmos de IM-05. A rota `/admin/projetos/:id` nunca foi declarada, portanto o detalhe pelo card precisa ser tratado como não confirmado.
- **Status/automação:** **cenário documentado**; não há teste das rotas administrativas.

