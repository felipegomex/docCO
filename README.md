# Central de Operação (Monorepo) — JCA Soluções

Portal interno unificado que reúne ferramentas, trilhas de conhecimento, cursos, gestão de acessos e o módulo de controle de projetos/implantação para a equipe de suporte.

---

## 📂 Estrutura do Monorepo

Este repositório (`JCASOLUCOES/CENTRALOPERACAO_DEPLOY`) é a fonte única do sistema, integrando frontend e backend.

```text
Central-Conhecimento-developer/
├── docs/                 # Documentação unificada (entrada: docs/README.md)
├── scripts/              # Scripts de utilidade (Deploy, Extração de metadados)
├── frontend/             # Angular 18 standalone (CSR; IIS publica `browser/`)
│   └── src/app/features/ # Módulos lazy (Gestor/executivo removidos em 24/09/2026)
└── backend/              # ASP.NET Core 8 (Web API)
    ├── Central_BackEnd/      # Controllers, Services, Models e EF Core
    └── Central_BackEnd.Tests/ # único projeto de teste (xunit) — `dotnet test`
```

---

## 🛠️ Stack Tecnológica

### Frontend
- **Angular 18** (Componentes standalone; CSR puro — IIS serve `browser/`)
- **Bootstrap 5 + ng-bootstrap 17**
- **Lazy Loading** em todas as rotas de features
- **JWT Auth** (Tokens de 4h em memória, Refresh em cookie HttpOnly)
- **Angular CDK Drag & Drop** para Kanban interativo

### Backend
- **ASP.NET Core 8** (Web API RESTful)
- **EF Core 8** (InMemory para desenvolvimento, SQL Server para produção)
- **`ICifrador`** para credenciais do módulo Acessos (AES-256-GCM pronto, hoje desligado)
- **Fila de auditoria em background** (`Channel` + `BackgroundService`) para o módulo Acessos
- **Rate Limiting** e **Brute Force Guard** integrados
- **Automatic team/role assignment** via Função/Perfil (Operadores)

---

## 🚀 Desenvolvimento Local

### 1. Requisitos
- Node.js 20+
- .NET SDK 8.0+
- Acesso à rede interna (SQL Server de homologação e AnythingLLM)

### 2. Rodando o sistema
Para subir ambos simultaneamente via CLI:
No diretório raiz, utilize o comando opencode **"subir interno"** ou execute manualmente:

**Backend:**
```powershell
cd backend/Central_BackEnd
dotnet run
```
Porta padrão: `http://localhost:1009` (Swagger disponível)

**Frontend:**
```bash
cd frontend
npm install
ng serve
```
Porta padrão: `http://localhost:4200`

**Ambiente Development:** o backend pode criar operadores de teste previsíveis; não publique nem reutilize essas credenciais fora do local.

**Homolog SQL:** usuários reais de `192.168.2.154` / `dbBUSINESS_HML`. ⚠️ **Esse banco é o de produção** — não existe staging; toda migration é mudança de schema em produção de primeira vez (ver [`docs/operacao/deploy.md`](./docs/operacao/deploy.md)).

**Interruptor:** `appsettings.Development.json` → `Database:UseSqlServer` (`true` = homolog, `false` = InMemory).

**Testes:** `scripts/validate.ps1` roda **os dois lados** no passo 5/6 (`dotnet test`) e 6/6 (`ng test --watch=false`, ChromeHeadless). O backend tem 944 casos (xunit) e o frontend 186 testes (Karma/Jasmine). Para rodar isolado:

```powershell
Set-Location .\backend\Central_BackEnd.Tests
dotnet test
Set-Location ..\..
Set-Location .\frontend
npx ng test --watch=false --browsers=ChromeHeadlessNoSandbox
Set-Location ..
```

O CI (`.github/workflows/ci.yml`) repete build, testes e o gate de segredos em PR para `developer` e `master`.

### 3. Fluxo ao concluir uma tarefa

```
subir interno  →  validar  →  git commit/push  →  validar  →  deploy limpo
   (testa)        (builda)     (grava)            (confere)    (IIS + smoke)
```

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
# se OK, revisar e stage somente os arquivos intencionais:
git status --short
git add -- <arquivos-intencionais>
git commit -m "fix: ..."
git push origin developer
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
# skill "deploy limpo" (ou etapas manuais em docs/operacao/deploy.md)
```

---

## ✨ Funcionalidades Atuais

### 🧭 Navegação reorganizada (sidebar, header, Home, JOTA)
- **Sidebar** (`header-nav.config.ts`): ordem — **Início** (Visão Geral `/` + Agenda + **Kanban Geral** `/kanban`), **SUPORTE** (resolver, fraseologia, modelos, trilhas), **Implantação** (Visão geral, Projetos, Kanban → redireciona para `/kanban`, Tarefas), **Ferramentas**, **Acessos Empresas** (*Empresas* → `/acessos/clientes`, com badge dinâmico do total de unidades via `AcessosBadgeService`; *Nova unidade* → `/acessos/clientes/novo`), **JCA - Administração Interna** (penúltima; Visão geral, Comercial, Administrativo, Financeiro, Onboarding, Procedimentos, Kanban ADM `roles: ['F']` → `/kanban`), **Conhecimento** (última). Os itens **Empresa** (`/empresa`, só redirecionava ao onboarding) e **Políticas** saíram da sidebar — `/politica` segue acessível só pelo link **Política Interna** do footer. Itens de menu podem ser escritos **por letra** de `PERFIL_ID` (`NavRole = PerfilUsuario | 'A' | 'S' | 'F' | 'P'`), imunes ao rename do rótulo. O Header tem o item de topo "Acessos das Empresas". A rota `/admin` redireciona para `/admin/cadastros`; não há rota ativa `/admin/dashboard`. **Módulo Gestor, Central Executiva `/executivo` e `/administrativo` removidos/desativados** (Gestor e `features/executivo/` excluídos em 24/09/2026 — `e87d763`). A seção fantasma **Administração** (sem filhos) saiu da sidebar: a tela **GESTORES** (gestores do módulo Ideias, por setor) é acessada pelo dropdown do header em **Perfil → Administração → Gestores** (`/gestores-ideias`)**
- **Header e Home:** busca unificada via `GlobalSearchComponent` e `BuscaIndexService`; o Header mantém “Pesquisar qualquer conteúdo...” e `Ctrl/Cmd+K`, enquanto a Home possui “Buscar na Central...” com resultados locais e sem redirecionar o foco
- **Home refeita**: saudação com nome, 6 acessos rápidos, "Continue de onde parou"/"Mais utilizados" (`RecentesService`, `localStorage cc.recentes.v1`, partem vazios) + agenda real de 7 dias
- **JOTA transversal:** FAB + painel em todo o `MainLayout`. O frontend usa `POST /api/rag-proxy/chat` relativo ao site e depende de reverse proxy/ARR não versionado; o backend pode retornar fallback simulado. Página `/chat` **removida** em 24/09/2026
- **Pós-login:** deep-link respeitado; sem deep-link → **`/`** (Home, qualquer perfil; Módulo Gestor removido)
- Detalhes por tela em [`docs/referencia/frontend.md`](./docs/referencia/frontend.md), APIs em [`docs/referencia/endpoints.md`](./docs/referencia/endpoints.md) e arquitetura em [`docs/arquitetura/componentes.md`](./docs/arquitetura/componentes.md)

### 🐛 Correção: Erro Visual da Agenda (Overlay/Backdrop)
**Problema**: Ao navegar para `/agenda`, a tela apresentava tom esbranquiçado/opaco com travamento de cliques (sidebar, header e área de conteúdo).
**Causa**: Retenção de overlay/backdrop no DOM + falha no ciclo de vida da rota (`ngOnDestroy` não limpava a camada de backdrop do `<body>`).
**Solução**: 
- Adicionado `position: relative` ao container `.agenda-shell`
- Limpeza automática de `.modal-backdrop` e classe `modal-open` do body no `ngOnDestroy`

### 🎯 Kanban Geral (quadro multi-setor) + Kanban Drag & Drop
- **Rotas**: `/kanban` (quadro), `/kanban/gargalos` (cards bloqueados com dias parado) e `/kanban/admin/colunas` (colunas, setores e leitura das etapas). `/implantacao/kanban` agora **redireciona** para `/kanban`; a rota `/administrativo` (Kanban ADM) segue desativada — o atalho ativo é o item "Kanban ADM" → `/kanban?perfil=F`.
- **Setor**: `tbsetor` (seed de 5: Suporte, Implantacao, Financeiro, RH, Comercial) e `SetorId` em operador, projeto, tarefa e coluna. Pills de setor na tela, com default no setor do usuário.
- **Permissão** (fonte da verdade: `SetorAuthorizationService`): qualquer um **visualiza**; só o **próprio setor edita**; `SE_ADMIN` e `PERFIL_ID` A/S/F editam qualquer setor; card **sem setor** só por super-usuário. A UI espelha a matriz e desabilita drag/menu quando não pode.
- **Tarefas e projetos no mesmo quadro** (`GET /implantacao/kanban/board?setorId=&tipo=&limite=`), com toggle Tarefas/Projetos/Ambos, card de projeto e aviso quando o quadro é truncado.
- **Bloqueio virou atributo**: a coluna "BLOQUEADO" está inativa e `tbtarefa.TRF_Bloqueada`/`TRF_MotivoBloqueio` foram removidas. O bloqueio é linha em `tbbloqueio`, com **motivo obrigatório**, 1 aberto por origem e aging por `BLQ_DataInclusao`; reabrir um item já bloqueado **atualiza o motivo** da linha aberta.
- **Etapas-template fixas** por setor (`tbetapsetor`): o CRUD existe na API, mas a tela só lista.
- **Drag & Drop**: alteração de status/coluna das tarefas via arrastar-e-soltar (`@angular/cdk/drag-drop`); abrir/fechar bloqueio **não move** o card.
- **Integração Automática**: Tarefas movidas para colunas "Reunião", "Treinamento" ou "Marco de Entrega" criam/atualizam eventos na Agenda (`tbagenda`) automaticamente
- **Backend:** `TarefaService.SincronizarAgendaAsync()` cria ou atualiza registros de `tbagenda`; a correspondência atual usa `ProjetoId` e título da tarefa, pois `AgendaItem` não possui `TarefaId`

### 📞 Histórico de contatos do Lead (Prospecção)
- **Funil comercial** (`/dashboard/comercial`): novos campos no lead (WhatsApp, E-mail, LinkedIn, Origem fixa com 8 canais) e **timeline de prospecção** com 7 tipos fixos (Envio de mensagem, Mensagem respondida, Sem resposta, Ligação realizada, E-mail enviado, Reunião agendada, Outro).
- **Automação**: registrar contato atualiza `LEAD_UltimoContato`; badge "aguardando resposta" (âmbar) no card quando o último tipo for "Envio de mensagem"; excluir contato recompõe `UltimoContato` com a maior data restante.
- **UI**: botão "Registrar contato" no card do Kanban + timeline completa no modal de edição do lead (tipo colorido, data, observação, excluir). O histórico **não move** o lead entre etapas (a progressão continua pelo drag & drop).

### 👁️ "Cliente viu o contato" — flag manual no Kanban
- **Flag**: `tblead.LEAD_ContatoVisualizado bit NOT NULL DEFAULT 0`, migration `20260928182047_LeadContatoVisualizado` (script em `Migrations/Sql/LeadContatoVisualizado_Idempotente.sql`). **Precisa ser aplicada no banco de produção antes do publish** — ver [`docs/operacao/deploy.md`](./docs/operacao/deploy.md).
- **Semântica**: marcação **manual** do responsável. Nenhum evento automático (e-mail enviado, contato registrado, resposta) alterna a coluna; o estado é independente do badge "aguardando resposta".
- **API**: `PATCH /api/v1/dashboard/comercial/leads/{id}/contato-visualizado` com `{ "visualizado": bool }`; o `PUT /leads/{id}` também aceita `contatoVisualizado`, aplicado só quando informado.
- **UI**: toggle "Viu / Não viu" no card (verde/cinza, com update otimista e rollback) + checkbox no modal do lead.

### 📉 Dashboard Integrado: 2 abas no Comercial e base de clientes no Administrativo
- **Comercial (`/dashboard/comercial`)**: a aba **Oportunidades foi removida** (redundante com a prospecção). Restam **Kanban de Leads** e **Propostas**; a rota antiga redireciona para o Kanban. A entidade `tboportunidade` e seus endpoints **permanecem** — a aba Propostas, o KPI comercial (`ValorPipeline`/`ValorFechado`) e o valor no card dependem deles. Como a tela removida era a única forma de criar oportunidade, a criação foi **movida para o modal do lead** (Valor estimado / Etapa / Observação, só na edição de um lead existente).
- **Administrativo (`/dashboard/administrativo`)**: mostra **só a base de clientes** (a tabela de contratos e o alerta de reajustes foram removidos). Restam 3 KPIs (clientes ativos, contratos ativos, licenças), a barra de filtros (campo de busca, Situação, Grupo, Ordenar por, Limpar) e a tabela única `# | Cliente | CNPJ | Situação | Licenças | Grupo` com contador do total de licenças da lista.
- **Backend**: novo `GET /api/v1/dashboard/administrativo/grupos` (`tbcliente_grupo`, novo espelho `ClienteGrupoLegado`) e `GET /administrativo/clientes` agora aceita `busca`, `campo`, `situacao`, `grupoId` e `ordenacao`, somando licenças em **uma** consulta com LEFT JOIN (cliente sem contrato continua na lista com `0`).

### 💰 Faturamento do Dashboard — DESLIGADO por decisão (interruptor)
- **Não é bug.** `FlagsDashboard.FaturamentoLiberado` (`backend/Central_BackEnd/Models/Dashboard/ConstantesDashboard.cs`) é `const bool = false` e desliga **todo** valor de faturamento e atraso derivado de `tbcontrato`.
- **Por quê:** o valor é uma **projeção** `QTDE_LICENCA × VALOR_UNITARIO`, não receita confirmada — o legado não tem coluna de valor total nem de data de faturamento (só `DIA_FATURAMENTO`, o dia do mês) e a regra de contrato vigente (`EhAtivo`) inclui `SITUACAO = 'Entregue'`, que é status de **entrega**, não de cobrança. **A informação ainda não foi liberada**, então a API devolve `0` (e lista de atrasos vazia) em vez de publicar número enganoso.
- **Como religar:** trocar `false` → `true`. **Do lado da API não há outra mudança** — sem migration, sem ajuste de DTO e sem mudança de rota; os campos continuam no JSON e `/financeiro/atrasos` volta a popular. Do lado da **UI**, os cards/tabelas precisam ser restaurados (ver o item seguinte).
- **Zerados por padrão:** `faturamentoMensalContratos` (`/administrativo/resumo`), `ContratoResumo.valorTotal`, `faturamentoPrevistoMes`, `faturamentoAtrasado`, `resultadoMes`, `contratosAtrasados` (`/financeiro/kpis`) e `GET /financeiro/atrasos` (lista vazia por retorno antecipado). Referência do que era exibido antes: **R$ 259.454,08**/mês e **R$ 163.970,03** atrasados em 68 contratos.
- **Continua funcionando:** reajustes (é **data**, não dinheiro), custos e orçamentos (`tbcusto`/`tborcamento`, tabelas que o sistema gerencia e escreve) e o gráfico de evolução.
- **Telas enxugadas junto:** a home perdeu o bloco "Financeiro" e os cards de Faturamento mensal / Reajustes 90d (e não chama mais `/financeiro/kpis`); o Administrativo perdeu os cards de faturamento e o alerta de reajustes (3 KPIs restantes); o Financeiro perdeu 4 KPIs e a tabela "Faturamentos em atraso" (restam Orçamentos aprovados pendentes, Reajustes em 90 dias, gráfico de evolução e o CRUD de custos/orçamentos). A coluna "Faturamento" dos Orçamentos **permanece** porque é `tborcamento.dataFaturamento` (dado nosso, não do legado).
- **Regra de negócio e diagnóstico:** [`docs/referencia/regras-negocio.md`](./docs/referencia/regras-negocio.md) e [`docs/operacao/troubleshooting.md`](./docs/operacao/troubleshooting.md). **Não "conserte" reativando sem contexto.**

### 🐛 Correções críticas do Dashboard Integrado (contra o banco legado)
Cinco bugs que derrubavam o módulo inteiro, com diagnóstico em [`docs/operacao/troubleshooting.md`](./docs/operacao/troubleshooting.md):
1. **Colisão de rotas no Comercial** — havia dois controllers com `api/v{version}/dashboard/comercial`, o que gerava `AmbiguousMatchException` (HTTP 500) em todas as rotas. O duplicado (`Controllers/Dashboard/ComercialController.cs`) foi apagado; o canônico é `Controllers/ComercialController.cs` (17 operações).
2. **`DATA_FATURAMAENTO` inexistente** — o espelho `TbContratoLegado` mapeava uma coluna que não existe. O real é `DIA_FATURAMENTO smallint` (dia do mês), agora `DiaFaturamento` (`short?`) em toda a cadeia. Uma coluna errada derrubava Administrativo **e** Financeiro.
3. **`QTDE_LICENCA` é `smallint`, não `decimal`** — mapear como decimal quebrava com `InvalidCastException` (Int16 → Decimal).
4. **`OPENJSON` incompatível** — o banco legado está em `compatibility_level 100` (SQL Server 2008), onde `OPENJSON` não existe, e o EF Core 8 o gera para `lista.Contains(x.Id)`. **Regra permanente do projeto: nunca filtrar em query por `Contains` de coleção local; agregar/carregar a tabela pequena e filtrar em memória** (ou usar `EXISTS`/subquery para chave única). O InMemory **não** reproduz esse erro.
5. **Reajuste sem data virava "0 dias"** — o alerta listava **101 falsos positivos** de 102 itens (contratos sem `DATA_PROXIMO_REAJUSTE` colapsados por `dias ?? 0`); só 1 era real (contrato 163 / GESTART, 23 dias). `ContratoResumo.diasParaReajuste` passou a ser **anulável** (`null` = sem data) e qualquer filtro precisa testar `!== null` / `HasValue`.

### 🔐 Acessos Empresas — módulo CRUD (`/acessos/clientes`)
- **Substituiu** a tela que lia uma planilha do Google Sheets (3 endpoints, somente leitura). Rota nova **top-level** `/acessos/clientes`; `/ferramentas/acessos` redireciona para lá.
- **3 níveis + 5 tipos**: grupo econômico (legado `tbcliente_grupo`, somente leitura — a antiga `tbgrupoacesso` foi dropada) → cliente/unidade (`tbclienteacesso`/`CAO_`, com `CAO_ClienteId` apontando para a empresa do ERP em `tbcliente`) → RDP, VPN, AnyDesk, Banco e Actyon Web (`tbacessordp`/`RDP_`, `tbacessovpn`/`VPN_`, `tbacessoanydesk`/`ADK_`, `tbacessobanco`/`BCO_`, `tbacessoactyonweb`/`AWB_`). **32 endpoints** em `/api/v1/acessos`, soft delete em cascata no cliente e flags `CAO_Tem*` recalculadas pelo serviço.
- **Nova unidade em 3 etapas (assistente roteado)**: `+ Novo Acesso` abre `/acessos/clientes/novo` — etapa 1 *Empresa* com typeahead (`GET /clientes/busca-erp`, min 2 caracteres) sobre `tbcliente` que **só preenche os campos**, etapa 2 *Tipos de acesso*, etapa 3 *Configurar*. O envio final é **um único `POST /clientes` com o agregado `acessos`**, transacional (empresa + acessos + flags + auditoria juntos) — se o CNPJ já tinha unidade, devolve a existente (`jaExistia: true`) em vez de duplicar. A **edição** da empresa continua em modal.
- **Senha só sai em 2 dos 32 endpoints**: `GET /rdp/{id}/senha` e `GET /vpn/{id}/senha`. Qualquer outro `GET` devolve apenas `temSenha: boolean`. Cada revelação enfileira `REVEAL_SENHA` em `tbauditoriaacesso` com usuário, IP e navegador — **nunca** a senha.
- **Gate de senha mantido**: o modal pede **usuário digitado + senha** (o backend confere o usuário contra o JWT — divergência = 401 auditada sem queimar tentativa); o ticket de `VISUALIZAR` vale 2 min, ou **5 min com "Lembrar"**, e o de `EDITAR` é de uso único. A senha revelada **auto-esconde em 30 s**.
- ⚠️ **Criptografia ATIVA no DI**: `CifradorAesGcm` (AES-256-GCM) grava as senhas novas; `CifradorEmBranco` é apenas fallback de teste e não está registrado. Linhas antigas sem o prefixo `v1:` continuam legíveis por compatibilidade até o backfill — `GET /acessos/estado-cifra` mostra a implementação, não a cobertura do backfill.
- ⚠️ **Sem segregação por papel**: `[Authorize]` na classe, sem policy. Qualquer operador autenticado gerencia o módulo; a restrição é por auditoria.
- ⚠️ **`tboperador.SENHA` é `varchar(10)` em texto claro e NÃO é coberto por esta cifra** — trabalho separado.
- ⚠️ **Armadilha de modelagem**: `SenhaCifrada` e `TemSenha` ficam **fora** de `AcessoBase`, em cada classe concreta, senão o `TemSenha` herdado leria a propriedade da base (sempre vazia) e devolveria `false`. Trávado por `TemSenhaTests`.
- Detalhes: regras em [`docs/referencia/regras-negocio.md`](./docs/referencia/regras-negocio.md), dados em [`docs/arquitetura/banco-dados.md`](./docs/arquitetura/banco-dados.md), telas em [`docs/referencia/qa-historias.md`](./docs/referencia/qa-historias.md).

### 👥 Cadastro de Operadores — Atribuição Automática por Função/Perfil
- **Sem seleção manual de equipe**: A equipe e papel são definidos automaticamente pela Função selecionada
- **Mapeamento**:
  - `FUNCAO_ID = 1` (Analista de Sistemas) → **Implantador** (claim `eh_implantador=true` no JWT)
  - `FUNCAO_ID = 2` (Suporte) → Atendimento operacional
  - `FUNCAO_ID = 3` (Programador) → Desenvolvimento
  - `FUNCAO_ID = 4` (Financeiro) → descrição de cargo; atribuído a quem tem `PERFIL_ID = 'F'`
- **Perfil não é função:** o rótulo do token vem de `PERFIL_ID` + `SE_ADMIN` (`A`→Administrador, `S`→Suporte, `F`→**Financeiro**, `P`→Produtor, resto→Usuario) e a **permissão do Kanban Geral é lida do `PERFIL_ID` cru**, nunca do rótulo nem de `FUNCAO_ID`. O rename `F`→"Financeiro" é só apresentação.
- **Migration relacionada:** `AddFuncao` (tabela `CC_Funcao` + `FUNCAO_ID` em `TBOPERADOR` + FK; script em `Migrations/Sql/AddFuncao.sql`). A tabela `CC_Funcao` foi fundida na legada `tbfuncao` pela migration `PadraoTabelasTb` (ganhou `CLASSIFICACAO` e `ATIVO`, e `FUNCAO_ID` foi widenada de `smallint` para `int`); `TBOPERADOR` passou a se chamar `tboperador`. A aplicação em cada ambiente deve ser confirmada pelo histórico de migrations

---

## 🚢 Deploy (Produção)

Pipeline em 5 etapas (detalhe: [`docs/operacao/deploy.md`](./docs/operacao/deploy.md)):

1. **Congelar** — working tree limpa, hash e branch registrados
2. **Validar** — `scripts/validate.ps1` (segredos + `dotnet build` + `ng build` + `dotnet test` + `ng test`)
3. **Empacotar** — `deploy.ps1 -BuildFrontend 1 -BuildBackend 1 -Publicar 0`
4. **Publicar** — `deploy.ps1 -BuildFrontend 0 -BuildBackend 0 -Publicar 1 -Backup 1`, somente se `BUILD_INFO` == HEAD
5. **Smoke** — `scripts/smoke.ps1` verifica frontend `:1010` e Swagger `:1009`

⚠️ **`deploy.ps1` não aplica migrations.** O banco é atualizado por scripts idempotentes em `backend/Central_BackEnd/Migrations/Sql`, aplicados à mão **antes** do publish: entre eles `KanbanGeral_Idempotente.sql` (setores/etapas/bloqueios; tem `DROP`, faça backup) e `FuncaoFinanceiro_Idempotente.sql` (função 4 = Financeiro, só cadastro). Detalhes em [`docs/operacao/deploy.md`](./docs/operacao/deploy.md).

**Defaults operacionais (confirmar no ambiente):**
- **Servidor Web:** `192.168.2.130` (IIS)
- **Banco de Dados:** `192.168.2.154` (SQL Server) — ⚠️ `dbBUSINESS_HML` **é o banco de produção**; não há staging
- **Portas:** Frontend `1010`, Backend `1009`

---

## 📝 Documentação Centralizada

Toda a documentação técnica reside em `docs/` (entrada: [`docs/README.md`](./docs/README.md)):

1. [`docs/README.md`](./docs/README.md) — entrada e matriz dos 12 documentos canônicos
2. [`docs/arquitetura/estrutura-projeto.md`](./docs/arquitetura/estrutura-projeto.md) — organização, backup e fluxo de trabalho
3. [`docs/arquitetura/visao-geral.md`](./docs/arquitetura/visao-geral.md) — contexto, stack e situação atual
4. [`docs/arquitetura/componentes.md`](./docs/arquitetura/componentes.md) — arquitetura, tokens e segurança
5. [`docs/referencia/regras-negocio.md`](./docs/referencia/regras-negocio.md) — regras de negócio
6. [`docs/arquitetura/banco-dados.md`](./docs/arquitetura/banco-dados.md) — dados, entidades e integrações
7. [`docs/referencia/endpoints.md`](./docs/referencia/endpoints.md) — APIs, métodos, corpos e autorização
8. [`docs/referencia/frontend.md`](./docs/referencia/frontend.md) — componentes, rotas e serviços frontend
9. [`docs/referencia/backend.md`](./docs/referencia/backend.md) — controllers, serviços e persistência backend
10. [`docs/referencia/qa-historias.md`](./docs/referencia/qa-historias.md) — histórias e cenários de QA
11. [`docs/operacao/troubleshooting.md`](./docs/operacao/troubleshooting.md) — diagnóstico e troubleshooting
12. [`docs/operacao/deploy.md`](./docs/operacao/deploy.md) — publicação, rollback e smoke

Auxiliares de operação: [`docs/operacao/setup-local.md`](./docs/operacao/setup-local.md) — setup da estação de trabalho (banco, segredos, subida) e [`docs/operacao/iis-setup.md`](./docs/operacao/iis-setup.md) — instalação e publicação no servidor IIS.

---

## 🤖 Assistência Opencode (Agents & Skills)

O projeto é otimizado para uso com assistentes de IA (opencode):
- **Skills:** `validar`, `deploy-limpo`, `subir-interno`, `frontend-design`.
- **Regras:** Definidas no arquivo `AGENTS.md`.

---

© 2026 JCA Soluções
