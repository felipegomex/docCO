---
id: sec.09-8-fluxos-integrados
titulo: "8. Fluxos integrados"
tipo: referencia
area: backend
sistema: central
tags: [backend, aspnet, controllers, services]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 3000
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/backend.md](../backend.md).

# 8. Fluxos integrados

### 8.1 Autenticação

```text
login/refresh
  -> AuthController
  -> IAuthService
  -> tboperador + tbrefreshtoken
  -> access JWT em resposta
  -> refresh em cookie HttpOnly cc_refresh
```

No refresh, o hash antigo é revogado e o novo recebe o hash em `SubstituidoPor`. Quando o logout chega com JWT válido, o controller revoga o cookie atual, apaga `cc_refresh` e `cc_lembrar` e devolve 204. O interceptor Angular atual não anexa Bearer a URLs `/auth/`; por isso a chamada de logout feita pelo frontend pode não alcançar essa revogação, especialmente quando o access expirou, embora o frontend limpe o estado local.

### 8.2 Acessos Empresas

```text
gate
  -> POST /acessos/clientes/{id}/autorizar   (id = 0 emite ticket de criacao)
  -> confere "usuario" digitado contra o JWT (divergencia = 401 auditada, sem
     queimar tentativa) e compara a senha no PasswordValidationService
     (sem cache) + BruteForceGuard (5 tentativas / janela de 5 min -> 15 min)
  -> AcessoTicketService.Emitir -> resposta traz o ticket (+validoPor), nao um "validado"
  -> header X-Acesso-Ticket nas operações sensíveis
     VISUALIZAR: 2 min (ou 5 min com "lembrar": true), reusavel na empresa
     EDITAR: 2 min, uso unico

leitura
  -> GET /acessos/clientes?page&pageSize   (Paginado<ClienteLista>, flags derivadas)
  -> GET /acessos/clientes/{id}            (5 listas + resumo, só temSenha)
  -> GET /acessos/estado-cifra             (implementacao + perdidos)

escrita
  -> POST/PUT/DELETE /acessos/{clientes|rdp|vpn|anydesk|banco|actyon-web}
  -> AcessosService: valida, grava, RecalcularFlagsAsync
  -> POST /acessos/clientes com "acessos" = CriarClienteComAcessosAsync
     (transação unica: valida itens, cria empresa + acessos, flags, cache,
      auditoria CREATE por registro — falha em qualquer item desfaz tudo)
  -> IAuditoriaFila.Enfileirar (CREATE/UPDATE/DELETE + JSON antes/depois)

revelação (só RDP e VPN)
  -> GET /acessos/rdp/{id}/senha | /acessos/vpn/{id}/senha
  -> auditoria REVEAL_SENHA enfileirada ANTES de decifrar
  -> ICifrador.Decifrar  ->  resposta com aviso de registro
  -> ProcessadorAuditoria (background, lotes de 50)
       -> tbauditoriaimplantacao  (CREATE/UPDATE/DELETE)
       -> tbauditoriaacesso      (REVEAL_SENHA)
```

A senha nunca volta em nenhum outro `GET`. O `antes`/`depois` da auditoria é montado com campos escolhidos e **exclui `SenhaCifrada`**; a troca de senha aparece só como observação. Falha de auditoria não desfaz a mutação.

### 8.3 Agenda

Eventos são armazenados com operador responsável, tipo, prioridade, SLA, projeto e participantes. Saídas convertem UTC para `America/Sao_Paulo`. Férias exigem período; Treinamento/Daily e Reunião/Atendimento têm regras específicas de horário, duração e recorrência. Sobreposição do mesmo responsável produz `ConflictException` e HTTP 409; a consulta de conflito não inclui a coleção de participantes.

### 8.4 Projetos, cards e Tarefas

Ao criar um Projeto, o controller chama `ProjetoService` e depois `ProjetoEtapaService.InicializarEtapasPadraoAsync`. Alterações de Tarefas recalculam a jornada e sincronizam cards com tarefas não arquivadas. Cards concluídos podem desbloquear o próximo (`Pendente` ou `Bloqueada` → `EmAndamento`, preservando `DataInicio`); a conclusão automática de card só ocorre quando o anterior está concluído; uma tarefa reaberta reabre o card afetado e as etapas posteriores `Concluida` passam a `Bloqueada` em cascata. Toda tarefa com projeto nasce e permanece ligada a um card do mesmo projeto: a criação sem etapa é recusada, a etapa de outro projeto é recusada e a atualização não move a tarefa entre projetos.

A sincronização por tarefas **recalcula também o `AtrasoDias` da etapa** (regra única de 4.2.2): antes só a edição manual (`AtualizarEtapaAsync`) recalculava, e o número que a tela lia ficava defasado por construção. A decisão de "atrasado" **sobe no contrato** (`ProjetoResumo.atrasado`, `TarefaResumo.atrasado`, `CardKanban.atrasado` + `CardKanban.dataConclusao`) — o **cliente consome** a decisão do backend; só o helper `ehAtrasada` recalcula, e apenas como **reserva** (quando o endpoint não traz o campo).

### 8.5 Chat e Conhecimento

O Database Explorer não usa o `AppDbContext`; abre a conexão configurada em `IDatabaseConnectionService` e consulta o banco alvo.

O Chat é servido **diretamente pelo AnythingLLM**, embutido em iframe na rota `/chat`. A Central só faz o SSO: `ChatSsoController` (`GET /api/chat/sso`) lê o `sub` do JWT, localiza o usuário correspondente em `GET {AnythingLLM:BaseUrl}/admin/users` (comparação de `username` com `OrdinalIgnoreCase`), emite token em `GET /users/{id}/issue-auth-token` e devolve `{ loginPath, url, token }`. O front (`ChatIframeComponent`) usa `apiRootUrl` + `/api/chat/sso` e carrega `url` no iframe. `AnythingLLM:InstanceUrl` define o host do iframe; sem ela, o host é derivado de `BaseUrl` removendo `/api/v1`. Erros: `401` sem identidade, `404` operador sem conta no AnythingLLM, `502` configuração ausente/falha e `504` timeout.

Os endpoints de conversa da Central (`/api/chat/agentes`, `/api/chat/perguntar` e `/api/chat/provedor`) foram removidos no commit `cb36ad3`, junto com `RagProxyController`, `ChatPerguntarService`, `ChatAgenteCatalogo` e `ChatAgenteStatusService`. As tabelas `tbchat_agente`, `tbchat_workspace_funcao`, `tbchat_provedor` e `tbchat_auditoria` continuam no schema, mas não são mais consumidas por endpoints de chat. O painel "Provedor de IA" do front, que ainda chamava essas rotas, foi removido em 2026-11-03.

O Conhecimento consulta documentos no Gitea configurado e exclui da árvore arquivos `INDEX.md` e symlinks. Revisores e workspaces são vinculados por administração. O sincronizador cria cada workspace AnythingLLM ausente na primeira execução **se a API instalada suportar a operação**; isso não cria vínculos de revisores. A migration/SQL semeia `tbchat_workspace_funcao` (funções 1, 2, 3 e 5), mas não semeia revisores.

Edição cria/atualiza branches e pull requests. Na aprovação, o token de serviço prepara a branch/alterações/PR e a credencial OAuth pessoal do revisor aprova e mescla. A rejeição também cria branch e PR com `status: rejeitado` no caminho original por token de serviço, e o revisor aprova/mescla com OAuth pessoal. Antes de usar rejeição, `_config/status-documento.yml` no repositório Gitea precisa permitir `rejeitado`. Conflito de SHA devolve SHA e conteúdo atuais para a UI comparar ou recarregar. A aprovação solicita sincronização; o endpoint manual administrativo é `POST /api/conhecimento/sincronizar` e o estado da Central é `GET /api/conhecimento/sincronizacao`. Na VM (`/opt/jca/sync/app`), o gatilho `POST /api/sync` e o estado `GET /api/status` usam a porta **8090** e o header `X-Central-Sync-Key` (`jca-sync-trigger.service`, bind `0.0.0.0`). O `sync.py` lê o mapa pasta→workspace de `_config/workspaces.yml` no Gitea (fallback para o mapa embutido), grava o metadado `docSource` com o caminho do arquivo e salva checkpoints parciais de localizações no manifesto fixo `/opt/jca/sync/state/manifest.json` durante a associação aos workspaces; `--workspaces a,b` limita a execução sem apagar o checkpoint dos demais.

### 8.6 Atendimento WhatsApp

```text
WhatsApp ──► OpenWA (gateway na VM 192.168.2.253:2785)
               │  webhook HMAC (POST /api/v1/whatsapp/webhook/openwa)
               ▼
             WhatsAppWebhookController  (lê o corpo CRU, limite 1 MiB)
               ▼
             WhatsAppWebhookService.ReceberAsync
               · módulo habilitado?        → não: 503
               · assinatura HMAC válida?   → não: 401
               · dedupe pela chave          → já existe: 200 { duplicado: true }
               ▼
             tbwhatsapp_webhook_fila  (gravado ANTES de processar)
               ▼
              WhatsAppWebhookProcessor (worker: lote 20, ciclo 2 s, 6 tentativas)
                · recupera as linhas presas em "processando"
                  (largada: todas; a cada 30 s: as de mais de 2 min)
                ▼
              WhatsAppWebhookService.ProcessarAsync
                · message.received → AtendimentoService.RegistrarEntradaAsync
                     · abre conversa/atendimento (contato NÃO é criado: manual)
                     · reabre o MESMO atendimento se estiver resolvido
                     · retoma o card em "Aguardando cliente" se o cliente respondeu
                     · aviso de sistema/status broadcast → DESCARTADO
                       (não cria contato, conversa, atendimento nem mensagem)
                · message.sent/ack/failed → atualiza o status da mensagem
                · session.*             → espelha o status CANÔNICO do número
                     (nunca o nome do evento) e grava o telefone

resposta do suporte
  POST /whatsapp/atendimentos/{id}/mensagens
    → AtendimentoService.EnviarMensagemAsync
        · trava a linha, revalida o estado
        · a PRIMEIRA resposta ATRIBUI o atendimento ao operador do token
        · grava a mensagem como "pendente" e fecha a transação
    → OpenWaClient.EnviarTextoAsync (POST /sessions/{id}/messages/send-text)
        · ok  → mensagem = enviada, com o messageId do gateway
        · erro→ mensagem = falhou, evento de sistema; a resolução não é desfeita
```

**Fila durável, não fila em memória:** o pool do IIS recicla, e uma fila em memória perderia mensagens exatamente no pior momento (redeploy). A fila vive no SQL Server e a entrega do gateway é *at-least-once* — por isso o reprocessamento é inofensivo (a unicidade `(ChatId, ExternalId)` da mensagem impede duplicata). Quando o processo morre no meio do processamento, a linha presa em `Processando` **volta para a fila** sozinha (`RecuperarLinhasPresasAsync`); para forçar na mão, `UPDATE tbwhatsapp_webhook_fila SET WAT_Estado = 0, WAT_Tentativas = 0, WAT_ProximoProcessamentoEm = NULL, WAT_Erro = NULL WHERE WAT_Estado = 1;` (procedimento em [`operacao/instalacao-openwa.md`](../../operacao/instalacao-openwa/INDEX.md) §9.4).

