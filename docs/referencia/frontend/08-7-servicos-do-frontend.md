---
id: sec.08-7-servicos-do-frontend
titulo: "7. Serviços do frontend"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas, componentes]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 3620
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 7. Serviços do frontend

Todos os serviços Angular de aplicação localizados usam `providedIn: 'root'`.

### 7.1 `core`

| Serviço | Responsabilidade | Dependências/estado |
|---|---|---|
| `AuthService` | Login, refresh single-flight, usuário, logout e estado de inicialização | `HttpClient`, `TokenStorageService`, `Router`; cookie HttpOnly |
| `TokenStorageService` | Access token e usuário somente em memória | Campos privados; sem local/session storage de token |
| `BuscaService` | Estado compartilhado de abertura, origem e consulta da busca | `BehaviorSubject` |
| `BuscaIndexService` | Índice local de páginas e acervos estáticos | Importa dados de ferramentas, cursos, SQL, procedimentos, utilitários, políticas e onboarding |
| `RecentesService` | Aprende rotas acessadas e ordena recentes/mais usados | `Router`, `NavigationEnd`, `localStorage` |
| `PapelUsuarioService` | Deriva `gestor` ou `operador` do perfil autenticado | `AuthService` |
| `PermissoesService` | Espelho de `GET /permissoes/minhas` em signals: `carregar`, `limpar`, `pode(recurso, acao)` e `podeRota(rota)`; alimenta o menu e o `permissaoGuard` | `HttpClient`, `ConfigService`, API `/permissoes`; `providedIn: 'root'` |
| `OperadoresService` | Lista operadores ativos | `HttpClient`, `/agenda/operadores` |
| `BuildInfoService` | Versão do build para exibição no Guia do Sistema: `carregar()` GET `/version.json` com fallback em `APP_VERSION`; cache em memória | `HttpClient`, `BuildInfo`; `providedIn: 'root'` |

### 7.2 Features

| Serviço | Responsabilidade | Fonte/dependências |
|---|---|---|
| `AcessosService` (`features/acessos/acessos.service.ts`) | Gate de senha (`autorizar` com `usuario`/`senha`/`lembrar`), estado da cifra, busca de empresas no ERP, grupos (somente leitura), clientes (inclui o POST com agregado `acessos`) e os 5 tipos de acesso, e os 2 endpoints de revelação | API `/acessos`; `providedIn: 'root'` |
| `AuditoriaService` (`features/auditoria/auditoria.service.ts`) | Leitura das duas trilhas (`listarAcessos` com `usuario?`/`inicio?`/`fim?`/`page?`/`pageSize?`, `listarImplantacao` com `entidade?`/`acao?`/`usuario?`/`inicio?`/`fim?`/`page?`/`pageSize?`); **não filtra por papel** — repassa a query e trata o `403` do backend | API `/auditoria`; `providedIn: 'root'`; tipos em `auditoria.model.ts` (espelham `Dtos/Auditoria/AuditoriaDtos.cs` em camelCase) sobre o `Paginado<T>` de `@shared/models/paginado.model.ts` |
| `CursosService` | Lista, categorias, busca e detalhe | Arrays estáticos de `cursos-novos.data.ts` |
| `FerramentasService` | Lista, categorias, busca e detalhe | `ferramentas.data.ts` (cada `Ferramenta` carrega `embutivel`, que decide iframe na Central x nova aba) + `texto.helper.ts` |
| `ProcedimentosService` | Lista, busca por setor e detalhe | `procedimentos.data.ts` + `texto.helper.ts` |
| `UtilidadesService` | Busca, favoritos e estatísticas de uso | `utilidades.data.ts`, normalização e `localStorage` |
| `AgendaService` | CRUD, movimentação, lote, tipos, funções e operadores | API `/agenda`; delega operadores a `OperadoresService`. Nas interfaces, `AgendaDetalhe`/`AgendaAtualizarRequest` carregam `recorrente`, `padraoRecorrencia`, `serieId` e `dataRepeticaoFim`; o modal preenche `Repetir até`/padrão na edição e o `PUT` envia `dataRepeticaoFim` (`undefined` quando vazio — não mexe no lote) e `padraoRecorrencia` |
| `ProjetosService` | CRUD de Projetos, cards, clientes, tipos e colunas; documentos por URL, sem upload binário | API `/implantacao/projetos` e serviços auxiliares |
| `TarefasService` | CRUD, Kanban, arquivamento, chamados, apontamentos e histórico; `minhasFuncoesGeridas()` (`GET /gestores/minhas-funcoes`) alimenta o `funcoesGeridas` do Kanban — gestor edita/move/exclui cards das funções que gerencia | API `/implantacao/tarefas`; delega operadores |
| `DashboardService` | KPI e agregados do dashboard de Implantação | API `/implantacao/dashboard` |
| `ColunasKanbanService` | **Catálogo de colunas compartilhado** (`listarCatalogo` → `?todas=true`), **visão do setor** (`listarQuadro(funcaoId)`, `obterVisao`, `salvarVisao`) e escrita do catálogo (`criar`, `atualizar`, `reordenar` — uma ordem só, para todos). **Não exclui coluna**: o caminho de saída é desativar | API `/implantacao/colunas-kanban` |
| `FuncoesService` ([`funcoes.service.ts`](../../../frontend/src/app/features/implantacao/services/funcoes.service.ts)) | **Kanban Geral**: lista de funções (`listar`), leitura das etapas da função (`listarEtapas`), quadro (`obterBoard`), movimento de card (`moverCard`), bloqueios (`abrirBloqueio`/`fecharBloqueio`) e gargalos (`listarGargalos`). Reexporta os tipos de [`funcao.model.ts`](../../../frontend/src/app/features/implantacao/models/funcao.model.ts). Foram removidos 3 métodos mortos que nunca tinham consumidor: `obter`, o trio `criar`/`atualizar`/`excluir` e `atribuirOperadores`, além de `obterBloqueio` | APIs `/implantacao/funcoes`, `/implantacao/etapas-funcao`, `/implantacao/kanban`, `/implantacao/bloqueios` e `/implantacao/gargalos` |
| `TiposProjetoService` | CRUD administrativo de tipos de Projeto | API `/implantacao/tipos-projeto` |
| `DatabaseService` | Metadados, busca, relações, procedures, triggers, query builder, comparação (schemas, bulk e procedures) e scripts de correção (`gerarScripts`, `gerarScriptsBulk`, `validarScript`, `compararProcedures`) | API `/database` |
| `ChatIframeComponent` | Página `/chat`: carrega o AnythingLLM em iframe autenticado por SSO; sem API REST de agentes e sem widget global (`JotaChatService` e `ChatContextoService` removidos no `cb36ad3`) | `GET /api/chat/sso`; ver seção 10.4 |
| `ConhecimentoService` | Permissão, árvore, documentos, salvamento, nova revisão, diff, histórico, aprovação/rejeição, OAuth e sincronização | `/api/conhecimento/*`; ver seção 10.4 |
| `ComercialService` | Leads (funil/kanban com `incluirArquivados`/`somenteArquivados`), **toggle de "cliente viu o contato"** (`definirContatoVisualizado`), **arquivar/desarquivar** (`arquivarLead`, `desarquivarLead`), **catálogo de produtos** (`listarProdutos`), oportunidades por produto (criação pelo checklist do modal do lead e ciclo de proposta) e histórico de contatos com nome (`listarContatosLead`, `criarContatoLead`, `excluirContatoLead`) | API `/dashboard/comercial` |
| `AdministrativoService` | Base de clientes com filtros (`FiltrosClientes`: `busca`, `campo`, `situacao`, `grupoId`, `ordenacao`), grupos econômicos e resumo. `listarContratos()` existe, mas **nenhum componente da UI o consome** (o alerta de reajustes foi removido junto com os cards de faturamento) | API `/dashboard/administrativo` |
| `FinanceiroService` | KPIs, atrasos, custos e orçamentos. `listarAtrasos()` foi **mantido** no serviço sem consumidor na UI: o endpoint existe e volta a responder quando o interruptor de faturamento for religado | API `/dashboard/financeiro` (somente `Administrador`) |
| `DashboardToastService` | Fila de avisos exibidos por `DashboardToastComponent` | State em memória; helpers de domínio em `shared/dashboard.utils.ts` |
| `SistemaService` (`features/sistema/sistema.service.ts`) | Cadastro de operadores (`listar` com `apenasAtivos`/`busca`/`funcaoId`, `criar`, `atualizar`, `alterarStatus`, `alterarSenha`) e catálogo de cargos (`listarCargos`) | API `/operadores` e `/cargos`; `providedIn: 'root'`; contratos `OperadorCriarRequest`/`OperadorAtualizarRequest` com **`cargoId` obrigatório** |
| `WhatsappService` (`features/whatsapp/whatsapp.service.ts`) | Fila de atendimentos, **Minhas Conversas**, painel do atendimento com **painel do cliente**, **histórico do cliente** (`historicoCliente`), **mídia** (`midiaBlob`), ações do ciclo de vida, números conectados, QR, métricas (abas **Geral**, **Qualidade**, **Clientes**), saúde do canal, **auditoria** (`listarAuditoria`), **configuração IA** (`obterIaConfig`, `salvarIaConfig`), **assistente IA** (`obterAssistenteIa`), o **vínculo do contato** (`fichaContato`, `empresasErp`, `vincularContato` — Fase 01 do novo ciclo), a **lista paginada de contatos** (`listarContatos(busca, pagina, tamanho)`) e o **iniciar conversa** (`iniciarConversa` — 2026-11-03), as **etiquetas/investigação/observação** (`listarEtiquetas`, `criarEtiqueta`, `atualizarEtiqueta`, `definirEtiquetasAtendimento`, `definirInvestigacao`, `definirObservacaoAtendimento` — Fase 06) e a **leitura administrativa de contatos** (`listarContatosAdmin`, `historicoContatoAdmin` — Fase 08) | API `/whatsapp`; `providedIn: 'root'`; tipos em `whatsapp.model.ts` (espelham `Dtos/WhatsApp/*.cs` em camelCase) |
| `WhatsAppRealtimeService` (`features/whatsapp/whatsapp-realtime.service.ts`) | **Fase 09**: conexão única do hub `/hubs/whatsapp`, streams dos seis eventos + `reconectado$`, entrada/saída nos grupos e `notificarDigitando(atendimentoId, modo)`. **Não é fonte da verdade**: recebe só IDs/status/metadados e a tela relê pela API | `@microsoft/signalr`, `AuthService` (token), `ConfigService`/`environment` (`whatsappHubUrl`, opcional; sem valor, deriva de `apiBaseUrl`); `providedIn: 'root'` |

### 7.3 Módulo Ideias, Melhorias e Erros

| Arquivo | Responsabilidade |
|---|---|
| `features/ideias/ideias.model.ts` | Contratos TypeScript espelhando `backend/Central_BackEnd/Dtos/Ideias/IdeiaDtos.cs`. Enums como `union type` com os valores numéricos do C# (`'Recebida'`, `'EmDiscussao'`…), nunca `number` |
| `features/ideias/ideias.constants.ts` | Rótulos, classes de badge, `CAMPOS_POR_TIPO` (os 3 formulários) e o mapa `OPERACOES_SUPORTADAS` + `resolverAcoes()` — ver §4.7 |
| `features/ideias/ideias.service.ts` | `listar`, `obter`, `minhasPermissoes`, `criar`, `atualizar`, `comentar`, `transicionar`, `definirPrioridadeOficial`, `arquivar`, `excluir` (DELETE admin ou gestor da ideia — geral ou da função do criador; apaga solicitação e satélites) |
| `features/ideias/gestores.service.ts` | `listar`, `resumo`, `salvar`, `excluir`. Serviço separado do `IdeiasService` de propósito: os endpoints de Ideias não exigem admin, e um filtro de admin neles seria regra errada |
| `features/ideias/ideias.shared.scss` | Parcial de estilos compartilhado (badges, campos, botões, avisos, modal). **Todo CSS usa variáveis CSS**, porque o sistema tem modo escuro (`[data-theme="dark"]`) e cor fixa quebraria o tema |
| `pages/lista` | `ListaIdeiasComponent` |
| `pages/nova` | `NovaIdeiaComponent` |
| `pages/detalhe` | `DetalheIdeiaComponent` |

Fora deste módulo, em `features/gestores/pages/gestores/`: `GestoresComponent` — a tela **GESTORES**, organizada por **setor** (`tbfuncao` via `GET /funcoes?apenasAtivos=true`), com contagem e lista de funcionários ativos (`GET /operadores?funcaoId=`, `SE_ATIVO='S'`), gestores de `GET /gestores` (+ `GET /gestores/resumo`), `POST /gestores` e `DELETE /gestores/{id}`. As duas cargas são **independentes** (sem `forkJoin` entre funções e gestores): `403` em `GET /gestores` liga o signal `semPermissao` e a tela vira **somente leitura** (aviso + botões de cadastrar/definir/excluir ocultos), sem esconder as funções; outros erros mantêm o erro bloqueante. O modal de gestor busca operadores ativos (`GET /operadores?apenasAtivos=true&busca=`) e marca os da própria função com badge "do setor" — o backend não exige que o gestor seja da função. Rota `/gestores-ideias` em `features.routes.ts`. Regra 2026-10-13: **no máximo um gestor ativo por função** — cadastrar outro substitui (o backend devolve `substituiuOperadorId/Nome`); remover é **excluir**, sem inativar.

**Pontos de registro** — o módulo aparece em 4 arquivos e esquecer um deixa a tela sem entrada:

| Arquivo | O que foi acrescentado |
|---|---|
| `features.routes.ts` | `path: "ideias"` com `loadChildren`; e `path: "gestores-ideias"` com `loadChildren` para o módulo `features/gestores` |
| `shared/config/header-nav.config.ts` | Item "Ideias, Melhorias e Erros" na seção Início **sem `roles`**. A tela GESTORES **não** está na sidebar: o acesso é pelo dropdown do header (**Perfil → Administração → Gestores**), visível só para `Administrador`; a seção fantasma "Administração" foi removida da sidebar |
| `core/services/busca-index.service.ts` | 2 entradas em `PAGINAS` (Ideias e Nova solicitação) + a entrada "Gestores" |

O item do menu **não tem `roles`** de propósito: quem não é gestor vê apenas as próprias solicitações, e esconder o item seria esconder trabalho que ele pode fazer.

**Regras que a implementação revelou** (confirme no código antes de mudar qualquer tela):

1. **Toda mutação substitui o estado local pela resposta.** `POST /transicoes`, `/mensagens`, `/prioridade-oficial`, `/arquivar` e `PATCH /{id}` devolvem o `IdeiaDetalhe` inteiro recem-lido. Não há update otimista — é o que impede status, histórico e mensagens de discordarem entre si.
2. **`IdeiaFiltro` não tem `Prioridade`.** Não há filtro de prioridade na listagem, porque a API não aceita o parâmetro. Filtro decorativo é pior que filtro ausente.
3. **Melhoria não tem campo "comportamento atual" no DTO.** O backend grava esse texto em `Problema`, então `CAMPOS_POR_TIPO.Melhoria` rotula a mesma coluna com "Funcionalidade ou processo afetado".
4. **Coluna "Criador" decide por `podeVerCriador`**, devolvido em `GET /ideias/minhas-permissoes` como `ehGestor || ehLeitor`. O leitor do setor CIAA também vê a coluna; o campo `criadorNomeParaGestor` continua sendo preenchido para gestor **e** leitor.
5. **Troca de tipo no formulário abre `confirm()`** listando os campos preenchidos que serão descartados: `IdeiaCriarRequest` guarda campos de Ideia e de Relato no mesmo registro.
6. **`FuncoesService.listar()`** (`/implantacao/funcoes`) é reusado para "Área relacionada" — não há endpoint duplicado no módulo.
7. **Excluir não é só admin.** No detalhe o botão aparece para `ehAdmin() || papel === 'Gestor'` (o papel vem do detalhe; 403 diz "Apenas administradores ou o gestor..."); na listagem o botão continua só para admin. Autor sem gestão não exclui — delete não tem volta (usar arquivar).

⚠️ **Limitação conhecida da tela de gestores:** o backend **não expõe troca de função** de um gestor já cadastrado. Para trocar, exclua e cadastre outro.

