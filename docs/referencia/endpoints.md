---
title: "Catálogo de endpoints"
description: "Índice do catálogo de endpoints, dividido por pasta de controllers, com contagens conferidas no código e fatos de estrutura."
date: "2026-10-30"
language: "pt-BR"
status: "canônico"
revisado_em: "2026-11-05"
---

# Catálogo de endpoints

O catálogo detalhado (contratos, autorização, respostas e limitações por controller) está organizado por **pasta da árvore `Controllers/`**; este arquivo é o índice. As contagens são de métodos de ação com atributo `[Http*]` no código atual, não de rotas Swagger ou de endpoints gerados pelo framework, e foram conferidas em 2026-10-18 pela contagem recursiva dos atributos na pasta `backend/Central_BackEnd/Controllers`, com a entrada `DocsController` (docs-api, fase 2) recontada em 2026-10-19, as colunas do Kanban (`ColunasKanbanController`) recontadas em 2026-10-20, o **módulo de atendimento WhatsApp** (`Controllers/WhatsApp`) recontado em 2026-10-27 (15 classes, 72 operações — Fase 01 do novo ciclo, ficha/typeahead/PATCH do vínculo do contato; Fase 06, etiquetas/investigação/observação; **Fase 07**, que só acrescentou parâmetros de filtro ao `GET /atendimentos`, sem rota nova; **Fase 08**, com duas leituras administrativas de contatos; **F6**, com `POST /atendimentos/{id}/visto`; e **Fase 3**, com `GET /contatos/{id}/foto`), recontado de novo em **2026-11-05** (**79** operações — entraram `GET /contatos/sem-contato`, `POST /contatos/vincular`, `GET /contatos/foto?chatId=` e `GET/PUT /configuracao/finalizacao-automatica`; recontagem geral do ciclo WhatsApp) e o **RBAC** (`PermissoesController`, `CargosController` e as 6 operações de `OperadoresController`) conferido em 2026-10-21, com o `PermissoesController` recontado em **2026-10-27** (5 operações, após `cargos-gerenciaveis` e a matriz).

## Índice e contagem

| Pasta | Classes | Operações `[Http*]` | Seção |
|---|---:|---:|---|
| `Controllers` (raiz) | 16 | 127 | [endpoints/raiz.md](endpoints/raiz/INDEX.md) |
| `Controllers/Implantacao` | 11 | 59 | [endpoints/implantacao.md](endpoints/implantacao/INDEX.md) |
| `Controllers/Dashboard` | 2 | 14 | [endpoints/dashboard.md](endpoints/dashboard.md) |
| `Controllers/Database` | 1 | 24 | [endpoints/database.md](endpoints/database.md) |
| `Controllers/WhatsApp` | 15 | 79 | [endpoints/whatsapp.md](endpoints/whatsapp/INDEX.md) |
| **Total** | **45** | **303** | — |

Duas classes são declaradas em arquivos de outro nome: `GargalosController` é a segunda classe de `BloqueiosController.cs`, e `DashboardController` (de implantação) é a segunda classe de `TarefasController.cs`; ambas estão em [`endpoints/implantacao.md`](endpoints/implantacao/INDEX.md).

## Versão, autorização e limite de requisições

- A API habilita versionamento com versão padrão `1.0`; as rotas dos controllers versionados são acessadas por `/api/v1`. A versão também pode ser informada pelo header `X-Api-Version`.
- **O hub SignalR `/hubs/whatsapp` não é API REST** e não entra na contagem de `[Http*]`: é notificação de saída do módulo WhatsApp (Fase 09). Os métodos do hub e a autenticação por `access_token` na query estão em [`endpoints/whatsapp.md`](endpoints/whatsapp/INDEX.md); a fonte da verdade continua sendo a API versionada.
- `JWT` significa `[Authorize]` com Bearer. `JWT + Administrador` significa `[Authorize(Roles = "Administrador")]` no método, combinado com a autorização do controller. `JWT + perm:<recurso>:<acao>` significa uma policy do RBAC resolvida por `PermissaoPolicyProvider`/`PermissaoHandler`; o 403 dessa policy é auditado (`ACESSO_NEGADO`).
- `Público` significa que não há `[Authorize]` no método/controller para a rota. `—` na coluna de limite significa que não foi encontrado `EnableRateLimiting` naquele controller/método; não significa que outra camada não possa proteger a aplicação.
- `login` usa uma janela fixa de cinco requisições por minuto fora de Development e cinquenta em Development, com chave pelo IP remoto; `validacao` usa cinquenta por minuto fora de Development e cem em Development; `leitura` usa cem por minuto. As políticas tentam identificar o usuário autenticado, mas `UseRateLimiter()` é registrado antes de `UseAuthentication()` no pipeline atual; por isso o limitador observa a requisição antes da autenticação e a chave efetiva é o IP remoto. A rejeição é `429 Too Many Requests`.

Fonte de referência: [Program.cs](../../backend/Central_BackEnd/Program.cs).

## Fatos confirmados por módulo

Contagens `[Http*]` por classe, conferidas em 2026-10-20 no código (chat recontado em 2026-11-05: **1 operação** — `GET /api/chat/sso`):

- **Raiz:** Auth 4; **Acessos Empresas 31**; **Auditoria 2**; Agenda 10; **Comercial 20**; ContextoProjeto 1; Docs 2; Funções (raiz) 3; Gestores 6; **Ideias 16**; IdeiasSeed 1; **Operadores 6**; **Permissões 5**; **Cargos 1**; Chat 1 (`GET /api/chat/sso`); Conhecimento 18.
- **Chat — iframe SSO (2026-11-03):** o único endpoint de chat é `GET /api/chat/sso`, JWT, que autentica a página `/chat`. O contrato anterior (`GET /api/chat/agentes`, `POST /api/chat/perguntar` e os três de provedor: `GET/PUT /api/chat/provedor`, `POST /api/chat/provedor/testar`) foi removido no `cb36ad3`.
- **Conhecimento — integrações (2026-10-30):** `GET /api/conhecimento/admin/integracoes` e `PUT /api/conhecimento/admin/integracoes/gitea`, ambos `JWT + Administrador` (`403` para não-admin). O GET responde `200` com `gitea_token_configurado`, `gitea_token_mascara` (`••••abcd`), `gitea_token_origem` (`banco`/`configuracao`/`null`), `cifra_configurada` e `chave_mestra_origem` (`banco`/`configuracao`/`null`) — **nunca** o token. O PUT recebe `{ "token": "..." }` e responde `204`; `400` para token vazio, fora de 16..255 caracteres ou com espaço; `503` quando a cifra não está disponível. O token fica cifrado em `tbconfiguracao_sistema` (`gitea.service_token`, AES-GCM) com fallback para `Gitea:ServiceToken`; a chave-mestra (`acessos.chave_cifra`) é gravada cifrada pelo DataProtection. Contrato em [`16-conhecimento-e-revisao-documental.md`](endpoints/raiz/16-conhecimento-e-revisao-documental.md).
- **Conhecimento — aprovação:** `POST /api/conhecimento/aprovar` aceita `pasta_destino` opcional; a escolha do revisor vence a `pasta_sugerida` do frontmatter e sem nenhuma das duas a resposta é `400`. O staging (`07-revisao/`, configurável) nunca é destino de aprovação.
- **Implantacao:** TiposProjeto 5; **ColunasKanban 6** (o `DELETE` de coluna saiu e os dois endpoints de visão do setor entraram); **Funções (Kanban Geral) 1; EtapasFuncao 6; Bloqueios 3; Gargalos 1; Kanban 2**; Projetos 17; Tarefas 16; Dashboard (implantação) 1; AdminDashboard 1.
- **Dashboard:** Administrativo 4; Financeiro 10.
- **Database:** Database Explorer 24.
- **WhatsApp (atendimento multiatendente):** Atendimentos 26 (as 20 anteriores + `PUT /etiquetas`, `PATCH /investigacao` e `PATCH /observacao` da Fase 06 + `POST /atendimentos/{id}/visto` da F6 + `POST /atendimentos/{id}/mensagens/contato` da Fase 7 do plano 2026-10). A **Fase 07 (Kanban Service Desk)** não criou rota nem mudou a contagem: acrescentou ao `GET /atendimentos` os filtros server-side `prioridade`, `empresaId`, `etiquetaId`, `emInvestigacao`, `primeiraRespostaSlaSituacao` e `resolucaoSlaSituacao` — aplicados **antes de contar e paginar**, com `400` para situação de SLA desconhecida; as **Fases 0/1 do plano** (2026-10-27) acrescentaram o filtro `protocolo` (igualdade exata) e os campos `protocolo`/`contatoTelefone` no resumo, sem rota nova; Números 10; Webhook 1 (**`[AllowAnonymous]`**, autenticado por HMAC); Métricas e saúde 3 (inclui a qualidade, só admin); Respostas rápidas 5; Configuração e monitor 4 (metas, monitor, **GET/PUT /configuracao/finalizacao-automatica**); SLA 4; Retenção de mídia 3; Contatos 10 (listar, duplicados, ficha, typeahead de empresas, PATCH do vínculo, unificar, foto, **sem-contato** e **vincular**); Grupos 3 (leitura, Fase C); Mídia 1; **Auditoria 1** (GET, só admin); **IA 3** (GET/PUT configuração, GET assistente por atendimento); **Etiquetas 3** (GET por qualquer atendente; POST/PATCH do catálogo só admin — Fase 06); **Admin 2** (`GET /admin/contatos` paginado com filtros server-side e `GET /admin/contatos/{id}/historico`, ambos só administrador — Fase 08; a escrita continua no PATCH da Fase 01). Categorias/Tags (Fase D) removidos. **A Fase 09 não acrescentou rota REST**: criou o hub SignalR `/hubs/whatsapp`, documentado em [`endpoints/whatsapp.md`](endpoints/whatsapp/INDEX.md). **A F6 acrescentou uma rota** (`POST /atendimentos/{id}/visto`) e a **Fase 3** acrescentou `GET /contatos/{id}/foto`; a **Fase 7 do plano 2026-10** acrescentou `POST /atendimentos/{id}/mensagens/contato`, levando a contagem a **79** (a Fase 6 não criou rota: vídeo `mp4`/`webm` e GIF entraram na whitelist do `POST /atendimentos/{id}/mensagens/midia` já existente).
- **Decisão de atraso no contrato (Implantação):** `ProjetoResumo`, `TarefaResumo` e `CardKanban` expõem `atrasado` — a **decisão do backend**, calculada por uma regra única de prazo (as telas que recebem o campo não recalculam). `CardKanban` passou a expor também `dataConclusao`, sem a qual o cliente não consegue avaliar a exceção "concluída dentro do prazo". Detalhes em [`endpoints/implantacao.md`](endpoints/implantacao/INDEX.md).
- **Recorte por setor (2026-10-22):** `GET /api/v1/implantacao/dashboard` aceita `funcaoId` e valida `PodeVerFuncao`; `GET /api/v1/implantacao/projetos` aceita `funcaoId` e o detalhe/etapas aplicam `PodeLerAsync`; `POST /projetos` aceita `FuncaoId` opcional validado por `PodeCriarFuncao`; `GET /implantacao/tarefas` devolve `403` para setor proibido. Nenhuma rota nova foi criada.

Notas dessas contagens:

- As classes usam rota versionada `/api/v1`; as exceções confirmadas são Chat e Conhecimento (rotas sem versão). A rota antiga `/api/rag-proxy` não é mais o contrato ativo; o contrato de chat vigente é `GET /api/chat/sso`.
- **RBAC:** `PermissoesController` (5), `CargosController` (1) e `OperadoresController` (6) usam `/api/v1`. As escritas de operador declaram policies `perm:aba.sistema.operadores:<acao>`; `GET /permissoes/recursos` exige `GESTOR_MASTER` e os demais `GET` de operador só exigem JWT. Em 2026-10-27 entraram `GET /permissoes/cargos-gerenciaveis` e a autorização própria de `GET`/`PUT /permissoes/matriz/{cargoId}` por `PodeGerenciarCargoAsync` (Master ou Tech Lead do setor/nível), sem policy `perm:*`.
- `ComercialController` tem **20 operações**: a contagem "17" do resumo do índice anterior ficou defasada em relação ao código.
- A pasta `Controllers` (raiz) tem **127 operações**: em 2026-11-05 saíram os 5 do contrato de chat (`/api/chat/agentes`, `/api/chat/perguntar`, `GET/PUT /api/chat/provedor`, `POST /api/chat/provedor/testar`) e entrou `ChatSsoController` com 1 (`GET /api/chat/sso`); a contagem inclui as **5** de `PermissoesController`, a de `CargosController` e as **6** de `OperadoresController`.
- `ContextoProjetoController` (rota base `/ia`) e `FuncoesController` (raiz, rota `/funcoes`) não constavam do índice anterior do catálogo; `IdeiasSeedController` e `OperadoresController` também não tinham linhas, e `GET /gestores/{id}` e `GET /gestores/resumo` existem no código sem linha no catálogo anterior. Os detalhes estão em [`endpoints/raiz.md`](endpoints/raiz/INDEX.md).
- O **Kanban Geral** tem **13 rotas** (1 de função + 6 de etapas da função + 3 de bloqueios + 1 de gargalos + 2 do quadro). Antes, com o eixo Setor, eram 18 (6 de setor + 6 de etapas-template + os mesmos 9).
- O módulo de Ideias está **implementado**: `IdeiasController` com 16 operações (incluindo o `DELETE /ideias/{id}`, admin ou gestor da ideia) e `GestoresController` com 6 — detalhes em [`endpoints/raiz.md`](endpoints/raiz/INDEX.md).
- O smoke não valida a API de negócio: [scripts/smoke.ps1](../../scripts/smoke.ps1) valida apenas a disponibilidade do frontend e a geração da definição OpenAPI (`swagger.json` com `openapi` e `paths` não vazio).
- Nenhum segredo, token real, senha real ou string de conexão foi incluído neste catálogo.
