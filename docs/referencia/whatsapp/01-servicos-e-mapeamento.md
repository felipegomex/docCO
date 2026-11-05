---
id: whatsapp.01
titulo: "Atendimento WhatsApp — serviços, componentes e mapeamento de edição"
tipo: referencia
area: atendimento
sistema: central
tags: [whatsapp, atendimento, servicos, backend, frontend, mapeamento]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - frontend/src/app/features/whatsapp
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2520
ficha: docs/modulos/whatsapp.md
---
> Parte de [Atendimento WhatsApp — implementação, contratos e operação](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## Serviços e lógica

Em `backend/Central_BackEnd/Services/WhatsApp`: `AtendimentoService` (+
`CicloVida`) — fila, envios, ciclo de vida, prefixo de autor
(`AplicarPrefixoAutorAsync`, reutilizado por `EnviarMensagemAsync` e
`EnviarEncerramentoAsync`), menções (`SubstituirMencoesAsync`) e reação
(`ReagirAsync`); `WhatsAppFinalizacaoAutomaticaService`
(`ArquivarCondicionalAsync`) — finaliza **e arquiva** na mesma rodada;
`WhatsAppMetricasService` — FCR; `OpenWaClient` — rotas do gateway
(`send-text`/`-image`/`-audio`/`-video`/`-contact`/`-document`) e
`messages/react` (`ReagirAsync`); `ContatoTelefoneResolver`
(`IContatoTelefoneResolver`) — telefone de `@lid` sob demanda;
`WhatsAppWebhookService` (`ProcessarReacaoAsync`, propaga `MentionedIds`);
`TelefoneNormalizador` (`Normalizar`, `NormalizarParaCartao`,
`FormatarExibicao`, `FormatarContato`); `WhatsAppVCard`;
`AtendimentoService.IniciarConversaAsync` — abre/reaproveita a conversa e o
atendimento ao "iniciar conversa" no Kanban, sem enviar mensagem e **sem criar
contato**. O contato **só nasce no vínculo**: `ContatoVinculoService.VincularAsync`
unifica o `PATCH /contatos/{id}` (`contatoId` informado → atualiza com lock
otimista) e a criação (`contatoId = null` + `chatId` → cria a linha em
`tbwhatsapp_contato`, usa o pushName quando o nome vem vazio, aponta as
conversas do chat e grava a empresa); o `POST /contatos/vincular` audita
`vincular` com a marca `contato_criado`. `RegistrarEntradaAsync` e
`SolicitarPrivadoAsync` só abrem/reaproveitam a conversa.
`WhatsAppGrupoService.ListarParticipantesAsync`
compõe o nome do integrante (`Participante.Nome` do gateway → contato salvo
pelo mesmo `ChatId` → telefone formatado) e persiste o nome descoberto.
Frontend: `whatsapp.service.ts` (`reagir`, `iniciarConversa`, `listarContatos`
paginado, leitura de `/contatos/sem-contato` e gravação de
`/contatos/vincular`), `WhatsAppRealtimeService` (`hubUrl`),
`ContatoAvatarComponent` (avatar por contato ou por `chatId`),
`whatsapp-contatos-painel` (busca, "Nova conversa com número" e lista paginada
com "Conversar"), `iniciar-conversa-modal` (destinatário e seletor de número),
`atendimento-modal-responsabilidade` (extraído da página e reutilizado no 409
do painel), `cliente-context-panel` e `contato-whatsapp.component.ts` (ficha e
busca para resolver `@lid`), `grupo-integrantes-modal` e o helper
`whatsapp.utils.ts` (`nomeOuTelefone`).

## Onde alterar

Backend em `backend/Central_BackEnd/Services/WhatsApp`, frontend em
`frontend/src/app/features/whatsapp`, testes em `backend/Central_BackEnd.Tests`.
Fluxo de dados da tela em
[10.7 Atendimento WhatsApp](../frontend/11-10-fluxos-de-dados/07-10-7-atendimento-whatsapp.md).

| Intenção | Backend | Frontend | Teste |
|---|---|---|---|
| “transição de status” | `AtendimentoService.CicloVida.AssumirAsync` | `pages/atendimento/atendimento-whatsapp.component.ts` | `WhatsAppAtendimentoTests.cs` |
| “enviar texto (negrito, Ctrl+V)” | `AtendimentoService.EnviarMensagemAsync` | `components/whatsapp-composer/whatsapp-composer.component.ts` (`aplicarNegrito`, `aoColar`) | `WhatsAppAtendimentoTests.cs` |
| “enviar mídia: imagem/áudio/vídeo mp4-webm/GIF” | `AtendimentoService.EnviarMidiaAsync` (whitelist + `Tipo`) e `OpenWaClient.EnviarMidiaAsync` (rota por MIME) | composer (`inputVideo`, ícone por tipo, `.wa-composer__progresso`) e `pages/atendimento` (anexo → contato → nota → texto) | `WhatsAppAtendimentoTests.cs`, `WhatsAppContratoGatewayTests.cs` |
| “enviar cartão de contato (vCard)” | `AtendimentoService.EnviarContatoAsync`, `OpenWaClient.EnviarContatoAsync`, `WhatsAppVCard.Gerar`/`NomeArquivo`, `TelefoneNormalizador.NormalizarParaCartao` | composer (mini-form `contatoPendente`) e `whatsapp-message.component.ts` (`ehContato`) | `WhatsAppVCardTests.cs`, `WhatsAppContratoGatewayTests.cs` |
| “autoria/negrito/card no balão” | — | `whatsapp-message.component.ts` (`segmentosNegrito`; card lê o rótulo, não a rota de mídia) | `whatsapp-message.component.spec.ts` |
| “modal de responsabilidade (409 → reenvio)” | `AtendimentoService.EnviarPelaCentralAsync`, `IniciarConversaAsync` | `components/atendimento-modal-responsabilidade` (extraído da página), `atendimento-whatsapp.component.ts` (`aoConfirmarAssumir`) e `whatsapp-contatos-painel` (409 do iniciar) | `atendimento-modal-responsabilidade.component.spec.ts` |
| “lightbox (imagem/vídeo/GIF)” | — | `midia-mensagem.component.ts` (`abrirLightbox`; render por MIME) | `midia-mensagem.component.spec.ts` |
| “webhook” | `WhatsAppWebhookService.ReceberAsync` | — | `WhatsAppWebhookTests.cs` |
| “número/QR” | `WhatsAppNumeroService.CriarAsync` | `pages/numeros` | `WhatsAppNumeroServiceTests.cs` |
| “SLA / métricas” | `SlaCalculo.MinutosUteis`, `WhatsAppMetricasService.CalcularAsync` | `pages/sla`, `pages/metricas` | `WhatsAppSlaTests.cs` |
| “busca server-side” | `AtendimentoService.AplicarFiltros` | `pages/minhas-conversas` (`aoDigitarBusca`) | `WhatsAppContatoTests.cs` |
| “reagir a uma mensagem (flag)” | `AtendimentoService.ReagirAsync`, `OpenWaClient.ReagirAsync`, `WhatsAppWebhookService.ProcessarReacaoAsync` | `whatsapp-message.component.*` (menu/chip), `whatsapp-conversation.component.*`, `atendimento-whatsapp.component.ts` (`aoReagir`) | `WhatsAppAtendimentoTests.cs`, `WhatsAppWebhookTests.cs`, `whatsapp-message.component.spec.ts` |
| “tipo de contato / telefone do contato” | `WhatsAppEnums.cs` (`TipoContatoWhatsApp`, `Rotulo`), `ContatoVinculoService` (`Enum.IsDefined`), `Dtos/WhatsApp/WhatsAppHistoricoDtos.cs` (`HistoricoContato.Numero`/`TelefoneNormalizado`/`TelefoneExibicao`) e `AtendimentoService` (preenche `TelefoneExibicao` com `FormatarContato`) | `whatsapp.model.ts` (`telefoneExibicao` em `ContatoFicha` e `HistoricoContato`), `vincular-empresa-modal.component.html` e `cliente-painel.component.ts` (prioridade `telefoneExibicao`, com fallback `numero`/`telefoneNormalizado`/`telefoneDoChat`) | `WhatsAppContatoTests.cs` |
| “telefone de `@lid` sob demanda” | `ContatoTelefoneResolver` (`IContatoTelefoneResolver`), `ContatoVinculoService.ObterAsync`, `AtendimentoService.RegistrarEntradaAsync` | `contato-whatsapp.component.ts` (chama `GET /contatos/{id}` quando `ehLid` e sem `telefoneExibicao`), `cliente-painel.component.ts` | `WhatsAppContatoVinculoTests.cs`, `WhatsAppContatoTests.cs` |
| “menção `@` do grupo vira nome” | `AtendimentoService.SubstituirMencoesAsync`, `EntradaWhatsApp.MentionedIds`, `WhatsAppWebhookService` | — | `WhatsAppGrupoAtendimentoTests.cs` |
| “prefixo `*NOME:*` nas respostas e no encerramento” | `AtendimentoService.AplicarPrefixoAutorAsync`, `EnviarMensagemAsync`, `EnviarEncerramentoAsync` | — | `WhatsAppAtendimentoTests.cs` |
| “finalizar sem enviar mensagem” | `AtendimentoService.FinalizarAsync`, `FinalizarAtendimentoRequest` (vazio) | `whatsapp-kanban.component.ts`, `kanban-service-desk-card.component.ts`, `whatsapp.service.ts` (ação direta, sem formulário) | `WhatsAppAtendimentoTests.cs` |
| “finalização automática (finaliza + arquiva)” | `WhatsAppFinalizacaoAutomaticaService.ExecutarAsync`/`ArquivarCondicionalAsync` | — | `WhatsAppFinalizacaoAutomaticaTests.cs` |
| “configurar a janela da automação” | `WhatsAppConfiguracaoController` (GET/PUT `configuracao/finalizacao-automatica`), `FinalizacaoAutomaticaConfig`, `IWhatsAppParametroService` | `pages/configuracao/configuracao-whatsapp.component.*`, `whatsapp.service.ts` (`obterFinalizacaoAutomatica`/`salvarFinalizacaoAutomatica`), `whatsapp.model.ts` (`FinalizacaoAutomaticaConfig`) | `WhatsAppConfiguracaoTests.cs` |
| “vincular empresa a chat sem contato (contato nasce aqui)” | `ContatoAdminService.ListarSemContatoAsync` (`GET /contatos/sem-contato`), `ContatoVinculoService.VincularAsync` (criação no `POST /contatos/vincular`; atualização no `PATCH`) | `pages/admin-contatos` (aba "Sem contato" + `vincular-empresa-modal`), `whatsapp.service.ts` | `WhatsAppContatoVinculoTests.cs`, `WhatsAppContatoTests.cs` |
| “abrir conversa pelo painel Contatos do Kanban” | `AtendimentoService.IniciarConversaAsync` (`POST /atendimentos/iniciar`), `ContatoUnificacaoService.ListarAsync` (paginado) | `whatsapp-kanban.component.*` (botão Contatos), `whatsapp-contatos-painel`, `iniciar-conversa-modal`, `whatsapp.service.ts` (`iniciarConversa`, `listarContatos`), `whatsapp.model.ts` (`IniciarConversaRequest`/`IniciarConversaResultado`) | `WhatsAppIniciarConversaTests.cs`, `whatsapp-contatos-painel.component.spec.ts`, `iniciar-conversa-modal.component.spec.ts` |
| “avatar e nome de contato/participante sem cadastro” | `WhatsAppFotoContatoService.ObterImagemPorChatAsync` (`GET /contatos/foto?chatId=`), `WhatsAppGrupoService.ListarParticipantesAsync` (`TelefoneExibicao`) | `ContatoAvatarComponent` (aceita `chatId`), `grupo-integrantes-modal`, `whatsapp.utils.ts` (`nomeOuTelefone`) | `WhatsAppGrupoAtendimentoTests.cs`, specs do modal |
| “buscar contato salvo no mini-form (vCard)” | `ContatoUnificacaoService.ListarAsync` | composer (`aoBuscarContatoSalvo`, debounce 300 ms, mínimo 2 caracteres; preenche nome/telefone/empresa) | `whatsapp-composer.component.spec.ts` |
| “hub apontando para a API do ambiente” | — | `whatsapp-realtime.service.ts` (`hubUrl`, `hubLocalInvalido`, `mesmaOrigem`; `conectar` aguarda o auth) | `whatsapp-realtime.service.spec.ts` |

## Armadilhas conhecidas

- Timestamp do gateway: use `WhatsAppTimestamp`. Webhook pode reenviar o evento;
  a fila garante idempotência.
- Cartão montado com a chave canônica (sem o 9) = número que não disca.
