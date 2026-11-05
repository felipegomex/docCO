---
id: 05-4-servicos-principais.01
titulo: "4.1 Autenticação, Acessos Empresas e Chat/Conhecimento"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 3303
ficha: docs/modulos/acessos.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### 4.1 Autenticação, Acessos Empresas e Chat/Conhecimento

| Serviço | Responsabilidade | Dependências | Validações confirmadas | Efeitos colaterais confirmados |
|---|---|---|---|---|
| `PermissaoService` (`IPermissaoService`) | **Motor único de autorização por recurso**: resolve ação/escopo, o conjunto efetivo do operador (`ObterMinhasAsync`), o bypass do Master (`EhMasterAsync`) e quem pode editar a matriz de cada cargo (`PodeGerenciarCargoAsync`/`ListarCargosGerenciáveisAsync`) | `AppDbContext`, `IGestorFuncaoService` | Operador inativo (`SE_ATIVO = "N"`) ou inexistente = negado; **negativa individual vence**; positiva individual vence o cargo; cargo nulo = negado (salvo exceção individual); escopo `Proprio`/`Funcao` recusa `funcaoAlvo` que não seja a própria função ou uma função gerenciada; `Tudo` ignora o alvo; `GESTOR_MASTER` (pelo `CARGO_Codigo`) = permitido com escopo `Tudo` e, em `ObterMinhasAsync`, todas as ações em todos os recursos ativos. Na matriz: Master gerencia tudo, exceto o próprio Master; Tech Lead só o mesmo `CARGO_FuncaoId` e nível inferior; demais negados | Cache **Scoped** por requisição (`operador|recurso|acao|funcaoAlvo` e lista de `minhas`), para o menu não repetir consultas; leitura `AsNoTracking`. `SubstituirMatrizAsync` desativa só `PEC_FuncaoId IS NULL` e audita a mudança |
| `OperadorAdminService` (`IOperadorAdminService`) | Cadastro e manutenção de operadores; catálogo de cargos | `AppDbContext`, `IAuditoriaImplantacaoService` | Regras da §3.10.1 (Master/Tech Lead, próprio setor, nível, autoproteção, senha ≤ 10, e-mail e ID) | Grava operador, audita `CREATE`/`UPDATE`/`ATIVAR`/`INATIVAR`/`SENHA` e liga `SE_ADMIN`/`PERFIL_ID` ao criar/promover Master |
| `AuthService` | Login, refresh, revogação e usuário | `AppDbContext`, `IConfiguration`, `BruteForceGuard` | Operador por id/email, ativo, senha SHA-256, refresh não revogado e não expirado | Cria hash de refresh no banco, revoga o anterior, grava substituição e emite JWT. O token carrega role e claim `perfil` (o **rótulo**), `eh_admin`, a claim **`perfil_id` com a letra normalizada** (`PerfilOperador.Normalizar`: trim + maiúsculas) e, quando existem, **`cargo_id`**, **`cargo_codigo`** e **`cargo_descricao`** — o cliente não precisa adivinhar a caixa com que o `PERFIL_ID` foi gravado e o `UsuarioResponse` também expõe os três campos do cargo |
| `BruteForceGuard` | Bloqueio em memória por chave | `ConcurrentDictionary` | Cinco falhas em cinco minutos bloqueiam por quinze minutos | Mantém contadores apenas no processo |
| `PasswordValidationService` | Gate de senha do módulo Acessos | `AppDbContext`, logger, `BruteForceGuard` | Operador **do token** (o corpo não escolhe a conta), ativo e senha via `SenhasIguais`; devolve `ResultadoSenha` | **Não guarda cache algum.** Zera o contador do `BruteForceGuard` no acerto e o incrementa na falha; não grava no banco |
| `AcessoTicketService` (`IAcessoTicketService`) | **Guarda a prova de senha** que o gate emite; o serviço de validação não guarda nada | `ConcurrentDictionary` em RAM, `System.Security.Cryptography` | Emite com `TtlVisualizar = 2 min` (ou `TtlVisualizarLembrar = 5 min` quando o pedido traz `lembrar: true`) e `TtlEditar = 2 min`; `UsoUnico` é verdadeiro para toda finalidade diferente de `VISUALIZAR`; recusa ticket de outro `OperadorId` ou de outro `ClienteAcessoId` | Ticket consumido (`ConsumidoEm`) permanece em memória até expirar, para responder "já usado" em vez de "inexistente"; varredura oportunista na emissão, sem timer dedicado |
| `AcessosService` | Clientes (criação/edição/soft delete), **busca de empresas no ERP** (`tbcliente`) e dos 5 tipos de acesso; leitura de grupos via `tbcliente_grupo`; revelação de senha; estado da cifra | `AppDbContext`, `ICifrador`, `IAuditoriaFila`, logger | Ver seção dedicada abaixo | Soft delete; recalcula flags; **enfileira** auditoria a cada mutação e a cada revelação |
| `ValidadoresAcesso` (`static`) | CNPJ, IPv4 e faixa de porta | — | Regras puras, sem I/O | Nenhum |
| `CifradorEmBranco` (`ICifrador`, **fallback de teste, NÃO registrado**) | Grava texto puro | logger | Nenhuma | `LogWarning` **uma vez por processo** |
| `CifradorAesGcm` (`ICifrador`, **ATIVO**) | AES-256-GCM com envelope `v1:iv:tag:ct` | `IConfiguration`, logger | Chave base64 obrigatória, base64 válido, exatamente 32 bytes — **lança** no construtor se falhar. Rejeita **marcadores de placeholder** (`TROQUE_POR_UMA_CHAVE`, `DEV_CHAVE_NAO_USAR`, `CHANGEME`, `CHANGE_ME`) **antes** do decode base64, com mensagem acionável. Expõe `GerarChaveBase64()` (CSPRNG, 32 bytes) | Cifra/decifra; retrocompatível com valor sem prefixo `v1:`; `LogInformation` no boot lembrando do backfill |
| `AuditoriaFila` (`IAuditoriaFila`, singleton) | Enfileira eventos de auditoria já serializados | logger | `Channel` limitado a 1000, `Wait` (não descarta), `SingleReader` | Incrementa `Perdidos` e loga `Error` se `TryWrite` falhar |
| `ProcessadorAuditoria` (`BackgroundService`) | Consome a fila em lotes de 50 e grava as duas tabelas de auditoria | `AuditoriaFila`, `IServiceScopeFactory`, logger | — | Cria o **próprio** `IServiceScope` por lote; trunca valores; falha só vira `LogError` |
| `AuditoriaLeituraService` (`IAuditoriaLeituraService`, scoped em `Program.cs`) | **Primeiro leitor** das trilhas (tela `/auditoria`): `ListarAcessosAsync` (filtro `usuario?`/`inicio?`/`fim?`) e `ListarImplantacaoAsync` (filtro `entidade?`/`acao?`/`usuario?`/`inicio?`/`fim?`), ambos paginados via `PaginadoDefaults.Normalizar` | `AppDbContext` | Consultas `AsNoTracking`, mais recentes primeiro | Somente leitura; nunca expõe senha |
| `ChatSsoController` | SSO passthrough do chat: autentica o operador e devolve a URL do AnythingLLM para o iframe | `IHttpClientFactory`, `IConfiguration`, logger | `[Authorize]`; exige `sub` no JWT e `AnythingLLM:BaseUrl`/`ApiKey` | `GET /api/chat/sso` devolve `{ loginPath, url, token }`; `401` sem identidade, `404` operador sem usuário no AnythingLLM, `502` falha/configuração e `504` timeout. Não persiste pergunta/resposta. |
| `ConhecimentoService` | Acesso por escopo e revisão de documentos Gitea | `AppDbContext`, `GiteaApiClient`, `ICifrador`, configuração, `IHttpContextAccessor`, serviço de sync | Admin ou revisor vinculado a função; paths e conteúdo validados; SHA base previne sobrescrita concorrente | `409` leva SHA/conteúdo atuais; aprovação e rejeição usam token de serviço para preparar branch/PR e OAuth pessoal para aprovar/mesclar. Rejeição atualiza `status: rejeitado` no caminho original; registra auditoria em `tbconhecimento_auditoria`; OAuth pessoal é cifrado em `tbconhecimento_credencial_gitea`. |
| `ConhecimentoSyncService` | Solicita job externo e consulta estado | `IHttpClientFactory`, configuração | Requer `KnowledgeSync:TriggerUrl` e `KnowledgeSync:Secret` | POST autenticado ao gatilho da VM; timeout/falha retorna estado pendente/erro, sem encerrar o job diário. |

#### Por que `AcessosService` audita por fila, e não inline

O módulo Implantação grava auditoria **inline**, via `AuditoriaImplantacaoService`, dentro da mesma requisição. Acessos Empresas usa a fila porque cada mutação produz **um evento a mais** que a operação de negócio (a revelação de senha é uma leitura que precisa de rastro) e porque serializar o estado antes de enfileirar é obrigatório: no consumidor a entidade de origem já foi descartada pelo `DbContext` da requisição, então serializar depois seria corrida sobre estado reciclado. A fila também desacopla o custo de auditoria da latência da tela — a mutação responde sem esperar a gravação.

⚠️ **Consequência aceita:** se o processo morrer com a fila cheia, o evento se perde. `BoundedChannelFullMode.Wait` faz a requisição esperar em vez de descartar em silêncio, mas **não** garante gravação. O contador `Perdidos` aparece em `GET /api/v1/acessos/estado-cifra` e cada perda gera `Error` no log. As duas vias coexistem de propósito: não unifique Acessos com `IAuditoriaImplantacaoService` sem avaliar o ganho (o service grava só em `tbauditoriaimplantacao`, e o módulo precisa também de `tbauditoriaacesso` para `REVEAL_SENHA`).

#### `AcessosService` — regras estruturais

- **Soft delete, nunca remoção física**, e `DELETE /clientes/{id}` inativa em cascata o cliente e os 5 tipos de acesso.
- `RecalcularFlagsAsync` roda após **toda** mutação de acesso e após editar o cliente: as 5 flags `CAO_Tem*` são derivadas e nunca aceitas do consumidor.
- **Grupos são somente leitura** (`tbcliente_grupo`): `ListarGruposAsync` lê a legada e não grava nada; `ValidarGrupoAsync` recusa `400` se o grupo informado não existir.
- `CriarClienteAsync` **exige `clienteId`** (empresa da `tbcliente`): sem ele, ou se a empresa não existir/estiver inativa no ERP, responde `400`. A chave de unicidade é **`CLIENTE_ID`** — se a empresa já tem unidade, devolve a existente com `JaExistia = true` sem duplicar (não há mais fallback por CNPJ nem cadastro manual). Identidade (fantasia, CNPJ, grupo) **não é copiada**: a tela lê da `tbcliente` por JOIN (`ResolverIdentidade`: `Fantasia` → `RazaoSocial` → `"Cliente {id}"`; sem vínculo → `"Unidade {id}"`).
- `CriarClienteComAcessosAsync` é a porta usada quando o `POST /clientes` traz o agregado `acessos`: valida cada item (`ValidarAcessosAgrupados` — servidor/RDP min. 3, usuário min. 3, senha min. 6, porta 1–65535 via `ValidadoresAcesso`), cria a empresa (se preciso) e todos os acessos **numa única transação** (`InserirAcessosAgrupadosAsync`, com cifragem), recalcula as flags, invalida o cache e audita um evento `CREATE` por registro; falha em qualquer item desfaz tudo.
- A revelação **enfileira a auditoria antes de decifrar**, mesmo sem senha cadastrada, e a resposta (`RevelacaoSenhaResponse`) carrega um `aviso` de que o registro existe.
- A troca de senha em `PUT` entra na auditoria como observação "senha trocada — valor não registrado": nem o antes nem o depois carregam `SenhaCifrada`.
- `Normalizar(string?)` centraliza "branco vira `null`", aplicado a todos os campos opcionais.
- `AtualizarClienteAsync` edita **apenas `Ativo` e `Observacao`**: identidade (fantasia, CNPJ, grupo) pertence à `tbcliente` e é ignorada no corpo da requisição (o operador corrige no ERP). Os snapshots de auditoria (antes/depois) usam `ResolverIdentidade`, nunca colunas locais — os helpers `ValidarGrupoAsync`, `ConverterGrupo` e `ValidarCnpjAsync` saíram junto com o cadastro manual.

O controller de Acessos chama `PasswordValidationService` com o ID do claim, no `POST /clientes/{id}/autorizar`: o corpo traz `usuario` (digitado no modal, conferido contra o JWT — divergência responde 401 auditada **sem** consumir tentativa do `BruteForceGuard`) e `senha`, de modo que a validação continua presa ao operador autenticado, não ao nome digitado. O que o operador recebe de volta não é um "validado" guardado, e sim o **ticket** emitido pelo `AcessoTicketService` (com `validoPor` em segundos).

