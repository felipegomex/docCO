---
id: 17-12-atendimento-whatsapp.17
titulo: "WA-109 — Telefone/@lid, menções, prefixo, tempo real e composer (rodada 2026-10-30) (+4)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias, telefone, lid, mencao, prefixo, signalr]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - backend/Central_BackEnd.Tests/WhatsAppContatoTests.cs
  - frontend/src/app/features/whatsapp/whatsapp-realtime.service.spec.ts
  - frontend/src/app/features/whatsapp/components/whatsapp-composer/whatsapp-composer.component.spec.ts
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 1800
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-109 — Telefone de cartão, `@lid` sob demanda e exibição (rodada 2026-10-30)

- **Objetivo:** o telefone exibido e o gravado não perderem o nono dígito, e o `@lid` só mostrar telefone quando houver resolvido.
- **Pré-condições:** número conectado; um contato privado (entrada por `@c.us`) e um contato `@lid`; para o `@lid`, uma sessão viva no gateway e outra rodada com a sessão parada.
- **Passos:** 1. Receber mensagem de um contato privado e conferir `WAT_Numero` no banco. 2. Abrir o atendimento e o painel do cliente; conferir o telefone exibido. 3. Receber mensagem de um `@lid` e conferir `WAT_Numero` e `WAT_TelefoneResolvido`. 4. Abrir a ficha do `@lid` com o gateway disponível. 5. Repetir com a sessão parada.
- **Resultado esperado:** no privado, `WAT_Numero` guarda o telefone de **cartão** (13 dígitos, com o 9) e `WAT_TelefoneResolvido` é gravado com `NormalizarParaCartao`; a tela mostra `telefoneExibicao` via `FormatarExibicao`/`FormatarContato`. No `@lid`, `WAT_Numero` fica **nulo** e `WAT_TelefoneResolvido` recebe o retorno de `GET sessions/{sessao}/contacts/{chatId}/phone`; com a sessão parada ou o gateway falhando, a ficha **abre normalmente** e a tela mostra `—` (sem erro). O resolvido não funde contato nem entra na duplicidade.
- **Estado atual/bloqueio:** a resolução é **sob demanda** pelo `ContatoTelefoneResolver`, acionado por `ContatoVinculoService.ObterAsync`; o cache é o próprio campo. O backfill de 2026-10-30 foi apagado no reset — não há telefone pré-gravado.
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/contatos/{id}` (dispara a resolução), `GET /api/v1/whatsapp/atendimentos/{id}/historico-cliente`; `tbwhatsapp_contato` (`WAT_Numero`, `WAT_TelefoneResolvido`).
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppContatoTests.cs` (`FormatarExibicao`, resolução sob demanda) e pelos specs do painel/`contato-whatsapp`.

### WA-110 — Menção `@` do grupo trocada pelo nome salvo (rodada 2026-10-30)

- **Objetivo:** a mensagem de grupo chegar legível, sem `@` seguido do JID cru.
- **Pré-condições:** grupo com um participante que já tenha contato salvo com nome; outro participante sem nome salvo; sessão conectada.
- **Passos:** 1. Enviar no grupo uma mensagem mencionando o participante com nome salvo. 2. Repetir mencionando o participante sem nome. 3. Abrir a conversa e o atendimento. 4. Conferir o payload bruto na fila do webhook.
- **Resultado esperado:** `@<jid>` vira `@NOME` (contato por `ChatId` ou `TelefoneNormalizado`; se não houver, participante do grupo por `ChatId`). Sem nome, vale o **telefone legível**; `@lid`/`@g.us` sem nome ficam **crus**. O texto persistido é o exibido na tela. O payload bruto **continua** na fila do webhook (nada é reescrito na origem).
- **Estado atual/bloqueio:** `EntradaWhatsApp.MentionedIds` é propagado pelo `WhatsAppWebhookService`; a substituição é `AtendimentoService.SubstituirMencoesAsync`. Sem `mentionedIds`, há fallback por `@<dígitos>` no corpo.
- **Rotas/endpoints/tabelas:** webhook `POST /api/v1/whatsapp/webhook/openwa`; `tbwhatsapp_mensagem`, `tbwhatsapp_webhook_fila`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppAtendimentoTests.cs` (menção por nome).

### WA-111 — Prefixo `*NOME:*` na resposta ao cliente (rodada 2026-10-30)

- **Objetivo:** o cliente saber quem respondeu, sem duplicar o prefixo.
- **Pré-condições:** operador com nome em `tboperador`; atendimento em “Em atendimento”.
- **Passos:** 1. Enviar uma resposta de texto. 2. Conferir o texto gravado e o recebido no WhatsApp. 3. Enviar outro texto que já comece com `*NOME:*`. 4. Enviar nota interna e uma mídia. 5. Repetir com operador sem nome cadastrado.
- **Resultado esperado:** o texto persistido **e** o enviado começam com `*NOME:* ` (primeiro nome em maiúsculas). Se o texto já começa com o prefixo, **não duplica**. **Nota interna e mídia não recebem prefixo.** Operador sem nome segue sem prefixo (não quebra o envio).
- **Estado atual/bloqueio:** a regra vive em `AtendimentoService.EnviarMensagemAsync` (`ObterPrimeiroNomeOperadorAsync`) e vale só para texto (`POST /atendimentos/{id}/mensagens`).
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/{id}/mensagens`; `tbwhatsapp_mensagem`, `tboperador`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppAtendimentoTests.cs` (prefixo, sem duplicar, nota/mídia sem prefixo).

### WA-112 — Hub no ambiente certo e composer com contato salvo (rodada 2026-10-30)

- **Objetivo:** o tempo real não apontar para HML/localhost em PROD e o composer buscar contato salvo.
- **Pré-condições:** runtime-config de PROD com `whatsappHubUrl` de HML (ou `localhost`) para exercitar o descarte; contatos salvos com nome/telefone/empresa.
- **Passos:** 1. Abrir o Kanban em PROD e conferir a URL do hub usada pelo navegador. 2. Recarregar a tela antes do token hidratar e conferir se a conexão ocorre depois. 3. Receber mensagem e conferir o Kanban sem esperar o polling. 4. No composer, abrir o mini-form de contato e buscar por nome/telefone/empresa. 5. Enviar o cartão de contato.
- **Resultado esperado:** o hub usa a **mesma origem** da `apiBaseUrl` (descarta `whatsappHubUrl` de outro ambiente/localhost) e `conectar()` tenta de novo após o auth. O Kanban assina `mensagem_recebida` e atualiza sem polling. O mini-form busca contatos salvos com debounce de **300 ms** e mínimo **2** caracteres e preenche nome/telefone/empresa; o envio do cartão segue o contrato de `POST /atendimentos/{id}/mensagens/contato`. O **botão de IA não existe** no composer (painel/admin de IA permanecem).
- **Estado atual/bloqueio:** `whatsapp-realtime.service.ts` (`hubUrl`, `hubLocalInvalido`, `mesmaOrigem`, `conectar`) e `whatsapp-composer` (`aoBuscarContatoSalvo`). Sem schema novo.
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/contatos?busca=`, `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/contato`; hub `/hubs/whatsapp`.
- **Status/automação:** **cenário derivado do código**; coberto por `whatsapp-realtime.service.spec.ts`, `whatsapp-kanban.component.spec.ts` e `whatsapp-composer.component.spec.ts`.
