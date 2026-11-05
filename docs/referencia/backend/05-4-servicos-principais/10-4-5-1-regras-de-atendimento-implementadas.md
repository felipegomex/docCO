---
id: 05-4-servicos-principais.10
titulo: "4.5.1 Regras de atendimento implementadas"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1683
ficha: docs/modulos/whatsapp.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

#### 4.5.1 Regras de atendimento implementadas

O contrato do módulo (`material-de-projeto/atendimento-whatsapp/README.md` §4) é o que o serviço implementa; o resumo:

| Regra | Como está implementada |
|---|---|
| **A primeira resposta atribui** | `EnviarMensagemAsync` trava a linha do atendimento, revalida o estado dentro da transação e, se não houver responsável, grava `atribuicao` com o operador do token. Duas respostas concorrentes não geram dois responsáveis: a segunda encontra o card já atribuído e leva `409` |
| **Assumir / transferir / devolver** | `AssumirAsync` sem responsável assume (status 1); com responsável de outro, responde `409` ("use transferir"). `TransferirAsync` exige o **destino**, valida que o operador existe em `tboperador` e grava `transferencia` com de/para. `DevolverAsync` solta o card e o devolve à fila (status 0) |
| **Resolver com mensagem de encerramento** | `ResolverAsync` aceita `EnviarEncerramento` (default `true`) e **sempre** zera `MensagensNaoLidas = 0` na **mesma transação** da resolução: `Status = 3`, `WAT_ResolvidoEm` e evento `resolucao`; com `EnviarEncerramento = false`, grava também o evento `mensagem_sistema` "Atendimento resolvido sem mensagem de encerramento.". A transação fecha antes de qualquer chamada ao gateway. Com `true`, a mensagem é enviada (texto configurado no número → texto do corpo → nada) e recebe o prefixo `*NOME:* ` do operador via `AplicarPrefixoAutorAsync` (sem duplicar); **falha no envio não desfaz a resolução** (evento `encerramento_falhou`, falha visível no card); com `false`, **nada** é enviado, nem a mensagem configurada no número. Depois do commit, `MarcarChatLidoSemFalharAsync` chama `IOpenWaClient.MarcarChatLidoAsync` (`POST sessions/{sessaoId}/chats/read` com `{ chatId }`) em best-effort: falha é logada e **não** reverte a resolução |
| **Reabrir** | Manual (`ReabrirAsync`) aceita **Resolvido ou Visto**; automático: mensagem recebida em atendimento resolvido **ou visto** reabre **o mesmo atendimento** (status 0, responsável nulo, contador de reaberturas++, `WAT_AbertoEm` = agora, primeira resposta e resolução zeradas). O histórico dos ciclos anteriores fica nos eventos. Desde a **Fase 2**, a reabertura automática é um **UPDATE condicional** (`Status IN (3, 6)`): se o job de finalização automática finalizou a linha no meio do caminho, o update afeta **0 linhas**, o atendimento é relido e, se estiver **Finalizado/Arquivado**, a mensagem abre um **novo atendimento** (via `CriarNovoAtendimentoAsync`, que centraliza a abertura) — nunca sobrescreve um Finalizado |
| **Aguardando cliente (status 2)** | Só entra a partir do status **1** e **mantém o responsável**. Sai por `retomar` (volta a 1 com responsável, 0 sem), por resolver, por devolver **ou porque o cliente respondeu** — no caso do cliente, o card volta a 1 **sem reiniciar o ciclo**. Transferir **não** tira o card da espera |
| **Finalização e arquivamento (F4)** | `FinalizarAsync` é **terminal e silencioso**: só aceita **Resolvido**, grava `Finalizado` e o evento **`finalizacao`** — **não envia mensagem ao cliente** (`FinalizarAtendimentoRequest` é vazio). `ArquivarAsync` (admin, só a partir de `Finalizado`) e o `WhatsAppArquivamentoJob` gravam **`arquivamento`**. O histórico passa a distinguir a ação real no lugar de reaproveitar o rótulo da resolução |
| **Finalização automática (Fase 2 do plano)** | `WhatsAppFinalizacaoAutomaticaJob` roda a cada **1 min** e chama `WhatsAppFinalizacaoAutomaticaService.ExecutarAsync`. A janela vem de `atendimento.finalizacaoAutomaticaMinutos` (**15** por padrão; **`<= 0` desliga a rodada**, sem alterar cards) e é editável em `GET`/`PUT /whatsapp/configuracao/finalizacao-automatica`: quem está **Resolvido** com `ResolvidoEm <= agora - janela` vai direto para **`Arquivado`**, em lote de **200**, sempre por `UPDATE` condicional (`ArquivarCondicionalAsync`) — a mensagem do cliente que reabre na mesma janela não é sobrescrita. **Grupo nunca finaliza sozinho** (`GrupoId` nulo **e** `ChatId` fora de `@g.us`). Só grava os eventos `finalizacao_automatica` **e** `arquivamento` se o UPDATE afetou exatamente **1 linha**; `OperadorId` é **nulo** (ação do sistema, `UsuarioInclusao = "sistema-finalizacao"`) e a descrição informa a janela usada. `atendimento_atualizado` (status `Arquivado`) + `fila_alterada` são publicados **depois do commit** e em best-effort |
| **Visto (F6)** | `MarcarVistoAsync` aceita **0/1/2**, grava `Status = 6`, zera `MensagensNaoLidas` e limpa responsável/`ResponsavelDesde`, com evento **`visto`**. Depois do commit, `MarcarChatLidoSemFalharAsync` faz o gateway marcar o chat como lido (`POST sessions/{sessaoId}/chats/read` com `{ chatId }`) em best-effort: falha (ou módulo desabilitado) é logada e **não** desfaz o Visto. Mensagem recebida em Visto reabre o mesmo card (`0`, evento `reabertura`). Métricas **não** contam Visto como resolvido; `emAndamento` continua 1/2 e a fila, 0; `WhatsAppNumeroService` não conta Visto como atendimento ativo |
| **Grupo por solicitante (F5)** | `RegistrarEntradaAsync` preenche `Conversa.GrupoId` e o **primeiro** `SolicitanteId` do ciclo, além de `Atendimento.SolicitanteId`; há **um atendimento ativo por grupo** e, após finalizar, nova mensagem abre outro com o novo solicitante. Listagem/detalhe preferem `Atendimento.SolicitanteId` (fallback `Conversa.SolicitanteId`); no resumo do Kanban, `SolicitanteNome` vem de `Atendimento.Solicitante.Nome` (fallback `Conversa.Contato.Nome`) e a busca textual também considera `Grupo.Nome` e `Solicitante.Nome`. `ObterOuCriarSolicitanteGrupoAsync` devolve `(GrupoId, ParticipanteId)`, atualiza nome do grupo/participante e telefone normalizado; `SolicitarPrivadoAsync` usa o `Contato` privado (não o `Participante`) e o `NumeroId` do grupo |
| **Nota interna** | Direção `2`, entra no histórico com evento `nota` e **nunca** passa pelo gateway |
| **Mensagem de dispositivo externo** | Mensagem com `fromMe = true` e **sem** `WAT_OperadorId` (veio do celular/PC) entra no histórico, mas **não** assume o atendimento |
| **Preview da mensagem de dispositivo externo** | `RegistrarEntradaAsync` atualiza `UltimaMensagemEm`/`UltimaMensagemTrecho` também quando `FromMe = true`, **sem** incrementar `MensagensNaoLidas`: antes o card mantinha o preview antigo até chegar uma mensagem do cliente, e a conversa parecia parada |
| **Mensagens órfãs e reentrega concorrente** | Ao abrir um ciclo, `AnexarMensagensOrfasAsync` vincula ao atendimento novo as mensagens da mesma conversa com `AtendimentoId` nulo (enviadas pelo celular antes de existir card) — sem isso elas existiam no banco e nunca apareciam na tela. `DbUpdateException` no índice único `(ChatId, ExternalId)` é tratada como **reentrega já processada** (`EntradaRegistrada(true, null, null)`), não como erro do webhook |

