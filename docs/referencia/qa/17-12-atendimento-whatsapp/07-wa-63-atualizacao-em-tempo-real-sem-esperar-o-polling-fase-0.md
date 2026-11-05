---
id: 17-12-atendimento-whatsapp.07
titulo: "WA-63 — Atualização em tempo real sem esperar o polling (Fase 09) (+6)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 3254
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-63 — Atualização em tempo real sem esperar o polling (Fase 09)

- **Objetivo:** mostrar no Kanban e no atendimento a mudança que acabou de acontecer, sem esperar o ciclo de 30 s.
- **Pré-condições:** módulo WhatsApp habilitado; backend com `AddSignalR()` e o hub `/hubs/whatsapp` mapeado; operador com `perm:aba.suporte.whatsapp:Ler`; duas sessões de navegador (ou uma no Kanban e outra no atendimento).
- **Passos:** 1. Abrir `/whatsapp` e `/whatsapp/atendimento/{id}` em sessões diferentes. 2. Na primeira, **enviar uma mensagem pela Central** (texto ou mídia) e, em seguida, assumir/transferir o card. 3. Observar a segunda tela **sem recarregar**. 4. Repetir com um ack de mensagem (`delivered` → `read`, e depois um `sent`/`ack` atrasado). 5. Provocar duas leituras concorrentes no atendimento (evento + mutação) e conferir que a resposta antiga é descartada.
- **Resultado esperado:** o backend publica `atendimento_atualizado`, `mensagem_recebida`, `mensagem_entregue`, `evento_registrado` e `fila_alterada` **depois do commit**; as telas recarregam o dado pela API imediatamente, e não no próximo polling. **A saída também é publicada:** o envio de texto/mídia pela Central e todas as transições (assumir, transferir, devolver, resolver, reabrir, aguardar/retomar, visto, nota, etiquetas, investigação, observação, prioridade/assunto, finalizar, arquivar, solicitar privado e nova conversa) disparam `atendimento_atualizado` + `fila_alterada` — os outros operadores veem a mensagem enviada sem esperar 30 s. **O ack não regride:** ack fora de ordem ou sem `status` textual não rebaixa o status; o servidor só avança e usa o campo numérico `ack` (0..4) como reserva. **A leitura antiga é descartada:** cada recarga tem `leituraSeq` e só a mais recente aplica o detalhe, então uma resposta em voo anterior à mutação não sobrescreve o estado novo. O payload traz só IDs/status/metadados — nenhum texto de mensagem, observação ou nota. Falha de publicação é logada e **não desfaz** a operação.
- **Estado atual/bloqueio:** o hub é **notificação de saída**; a fonte da verdade continua sendo a API. O polling de 30 s permanece como fallback e reconciliador. Sem o hub (SignalR desligado), a tela volta ao comportamento antigo de 30 s (WA-67). A gravação da conversa é **at-least-once**: reentrega concorrente barrada pelo índice único `(ChatId, ExternalId)` é tratada como processada, e mensagens órfãs da conversa são anexadas ao ciclo recém-aberto.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`; hub `/hubs/whatsapp` (métodos `EntrarNoAtendimento`/`EntrarNaFila`/`EntrarNoOperador`); `GET /api/v1/whatsapp/atendimentos`; `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; `WhatsAppRealtimeTests` cobre publicação pós-commit e payload sem conteúdo sensível; `WhatsAppSincronizacaoTests` cobre ack monotônico, ack numérico, `message.sent` sem rebaixar, envio publicando depois do commit, preview de `FromMe`, órfãs anexadas e o timestamp tolerante; `atendimento-realtime.component.spec.ts` cobre a resposta antiga que não sobrescreve a leitura mais recente, e `whatsapp-kanban.component.spec.ts` cobre a recarga por evento.
### WA-64 — Reconexão do hub e recarga silenciosa (Fase 09)

- **Objetivo:** o operador não perder eventos quando a rede oscila e a tela voltar sozinha, sem F5.
- **Pré-condições:** Kanban ou atendimento aberto com o hub conectado; rede instável ou serviço do backend reiniciado.
- **Passos:** 1. Abrir `/whatsapp`. 2. Derrubar a conexão (reiniciar o backend ou simular perda de rede). 3. Restabelecer. 4. Mudar um atendimento em outra sessão.
- **Resultado esperado:** o `WhatsAppRealtimeService` tenta reconectar com `withAutomaticReconnect([0, 2, 5, 10, 30] s)`; se a conexão fechar de vez, aplica **backoff exponencial próprio** (1 s, 2 s, 4 s… até 30 s). Ao reconectar, **reentra nos grupos** de atendimento, fila e operador e dispara `reconectado$`, que faz a tela recarregar em silêncio. Uma conexão por aplicação; `conectar()` é idempotente.
- **Estado atual/bloqueio:** eventos perdidos durante a queda são recuperados pela recarga de `reconectado$` e pelo polling; o serviço não reproduz o histórico do hub. Se o token expirar, o `accessTokenFactory` usa o token atual em memória, e a próxima tentativa pode falhar até o refresh acontecer — o polling cobre esse intervalo.
- **Rotas/endpoints/tabelas:** `/whatsapp`; hub `/hubs/whatsapp`; `GET /api/v1/whatsapp/atendimentos`; sem tabela nova.
- **Status/automação:** **cenário documentado**; `whatsapp-realtime.service.spec.ts` cobre reconexão, reentrada de grupos, `reconectado$`, backoff e `desconectar`.
### WA-65 — Indicador interno de digitação aparece e expira em 5 s (Fase 09)

- **Objetivo:** avisar que outro operador está digitando no mesmo atendimento, sem revelar conteúdo.
- **Pré-condições:** dois operadores no mesmo atendimento; ambos com o hub conectado.
- **Passos:** 1. O operador A abre o atendimento e começa a digitar. 2. O operador B observa o cabeçalho. 3. A para de digitar e B espera. 4. B começa a digitar.
- **Resultado esperado:** B vê o indicador após o evento `digitando` (payload só com `atendimentoId` e `operadorId`) e ele **expira em 5 s** sem novo evento. A tela de A **ignora** o próprio `operadorId` e não mostra o indicador para si. O evento interno é publicado mesmo em modo `nota`.
- **Estado atual/bloqueio:** o indicador é **efêmero** e não é persistido; a ausência dele não é erro. O componente usa `TYPING_EXPIRA_MS = 5000`; o backend publica `digitando` no grupo `atendimento-{id}`.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; hub `/hubs/whatsapp` (`Digitando`); sem tabela.
- **Status/automação:** **cenário documentado**; `atendimento-realtime.component.spec.ts` cobre aparição, expiração em 5 s e o descarte do próprio operador; `WhatsAppRealtimeTests` cobre autorização e payload do typing.
### WA-66 — Nota interna não gera typing externo; mensagem gera (Fase 09)

- **Objetivo:** o cliente do WhatsApp só vê "digitando" quando o operador está escrevendo uma **mensagem**, nunca uma nota interna.
- **Pré-condições:** atendimento com sessão do gateway conectada; operador autorizado a escrever.
- **Passos:** 1. No atendimento, alternar o composer para **nota interna** e digitar. 2. Conferir no gateway/celular do cliente que **não** há presença. 3. Alternar para **mensagem** e digitar. 4. Conferir a presença no cliente. 5. Repetir a digitação em menos de 3 s.
- **Resultado esperado:** em `modo = "nota"`, `WhatsAppTypingHandler` publica apenas o evento interno e **nunca** chama o gateway. Em `modo = "mensagem"`, publica o interno e chama `POST /api/sessions/{sessionId}/chats/typing` com `{ "chatId": "...", "state": "typing" }` (OpenWA 0.25; estados aceitos `typing`, `recording` e `paused`). O debounce de **3 s** por atendimento+operador suprime repetições tanto no frontend quanto no backend. Falha de presença é logada em `Debug` e não vira erro para o operador.
- **Estado atual/bloqueio:** o gateway documenta **10 req/1 s, 100 req/60 s e 1000 req/1 h por IP**; o debounce mantém o consumo dentro da cota. A presença é sinal efêmero — não há histórico nem persistência.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; hub `/hubs/whatsapp` (`Digitando(atendimentoId, modo)`); gateway `POST /api/sessions/{sessionId}/chats/typing`; `tbwhatsapp_atendimento`, `tbwhatsapp_conversa`, `tbwhatsapp_numero` (leitura).
- **Status/automação:** **cenário documentado**; `WhatsAppRealtimeTests` cobre a rota/corpo do gateway, o modo `nota` que não chama o gateway e o debounce; `atendimento-realtime.component.spec.ts` cobre o debounce do frontend.
### WA-67 — Polling continua funcionando com o SignalR desligado (Fase 09)

- **Objetivo:** garantir que a ausência do hub degrada a experiência, mas **não quebra** as telas.
- **Pré-condições:** backend sem `MapHub` (ou hub inacessível), módulo WhatsApp habilitado; operador com acesso.
- **Passos:** 1. Abrir `/whatsapp`, `/whatsapp/minhas-conversas` e `/whatsapp/atendimento/{id}`. 2. Confirmar no console/network que a conexão do hub falha. 3. Mudar um atendimento em outra sessão. 4. Esperar até 30 s.
- **Resultado esperado:** nenhuma tela mostra erro fatal; `conectar()` falha silenciosamente (ou agenda reconexão) e o **polling de 30 s** continua atualizando Kanban, lista e atendimento. Os indicadores de digitação simplesmente não aparecem. A tela de números mantém o ciclo próprio de 15 s.
- **Estado atual/bloqueio:** é o comportamento de **fallback por desenho**: o SignalR só acelera. O polling pausa com a aba oculta e recarrega ao voltar; em fila grande, o card pode levar até 30 s.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/minhas-conversas`, `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/atendimentos`; sem tabela nova.
- **Status/automação:** **cenário documentado**; `whatsapp-realtime.service.spec.ts` cobre a tentativa de conexão sem token/plataforma e a falha silenciosa; `atendimento-realtime.component.spec.ts` cobre `notificarDigitando` sem conexão sem quebrar a tela.
### WA-68 — Skeleton e erro com retry em blocos independentes (Fase 10)

- **Objetivo:** o operador nunca ficar sem retorno e a falha de um bloco não derrubar os demais.
- **Pré-condições:** módulo WhatsApp habilitado; atendimento com histórico do cliente; gestão de contatos com registros; navegador sem browser de validação automatizada nesta sessão (execução manual em HML).
- **Passos:** 1. Abrir `/whatsapp/atendimento/{id}` e observar a carga inicial. 2. Simular falha do detalhe (rede/rota) e conferir a mensagem e o botão **Tentar de novo**. 3. Simular falha apenas do histórico do cliente com o detalhe carregado. 4. Repetir 1–3 no Kanban e em `/whatsapp/admin/contatos`. 5. Provocar `401` e `403` e conferir as mensagens.
- **Resultado esperado:** durante a carga aparece **skeleton** no bloco (`.wa-atendimento__esqueleto`, `.wa-kanban__esqueleto`, `.wa-contatos__esqueleto`), sem spinner de página inteira nem salto de layout. O erro mostra mensagem em PT-BR e **"Tentar de novo"**; `401` pede nova sessão e `403` informa falta de permissão, **sem expor código HTTP**. A falha do histórico do cliente **não** derruba o painel do atendimento; durante a recarga silenciosa o indicador **"atualizando…"** (`.wa-atualizando`) aparece de forma discreta. O vazio orienta o próximo passo.
- **Estado atual/bloqueio:** implementação coberta por specs (`atendimento-estados.component.spec.ts`, `whatsapp-admin-contatos.component.spec.ts` e Kanban), mas a **verificação visual não foi executada** nesta sessão; os cenários dependem do ambiente publicado.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/admin/contatos`; `GET /api/v1/whatsapp/atendimentos`, `GET /api/v1/whatsapp/atendimentos/{id}`, `GET /api/v1/whatsapp/atendimentos/{id}/historico-cliente`, `GET /api/v1/whatsapp/admin/contatos`; sem schema novo.
- **Status/automação:** **cenário a executar em HML — não é aceite**. A existência dos estados é comprovada por spec; o comportamento visual e a mensagem final dependem da execução manual.
### WA-69 — Layout em 1440/1024/390 px com uma área por vez no mobile (Fase 10)

- **Objetivo:** o Service Desk responder nos três breakpoints sem esconder a conversa nem quebrar o layout.
- **Pré-condições:** atendimento com cliente, conversa e painel de operação preenchidos; navegador com redimensionamento manual ou DevTools.
- **Passos:** 1. Abrir `/whatsapp/atendimento/{id}` em **1440 px** e conferir as três colunas. 2. Reduzir para **1024 px** e conferir conversa + uma coluna lateral, alternando **Cliente/Operação**. 3. Reduzir para **390 px** e conferir uma área por vez. 4. Repetir em **1920, 1366, 1024, 768 e 390 px**, nos temas claro e escuro.
- **Resultado esperado:** **≥ 1440 px** três colunas; **768–1439 px** conversa sempre visível + **uma** coluna lateral por vez, com o seletor Cliente/Operação; **< 768 px** uma área por vez, header compacto e compositor **sticky** na conversa. As colunas recolhíveis continuam funcionando no desktop. Não há sobreposição, corte de conteúdo nem rolagem horizontal indevida na página de atendimento.
- **Estado atual/bloqueio:** os breakpoints estão no SCSS (`atendimento-whatsapp.component.scss`); a matriz visual **não foi executada** nesta sessão. Em 768×1024 (tablet) o comportamento esperado é o de uma coluna lateral por vez, e não o mobile.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; sem endpoint novo; sem schema.
- **Status/automação:** **cenário a executar em HML — não é aceite**. Há spec da troca de área, não da matriz de larguras.
