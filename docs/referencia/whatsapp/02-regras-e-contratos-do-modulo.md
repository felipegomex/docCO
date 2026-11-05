---
id: whatsapp.02
titulo: "Atendimento WhatsApp — regras de implementação e contratos"
tipo: referencia
area: atendimento
sistema: central
tags: [whatsapp, atendimento, regras, contratos, enums, midia, contato]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - frontend/src/app/features/whatsapp
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 3660
ficha: docs/modulos/whatsapp.md
---
> Parte de [Atendimento WhatsApp — implementação, contratos e operação](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).
> A norma de negócio canônica está em [Regras de negócio — 16](../regras-negocio/16-modulo-de-atendimento-whatsapp-atribuicao-ciclo-de-vida-e-ag.md) e [25](../regras-negocio/25-atendimento-whatsapp-protocolo-foto-telefone-e-schema.md); o contrato HTTP, em [Endpoints WhatsApp](../endpoints/whatsapp/INDEX.md), [17](../endpoints/whatsapp/17-observacoes-de-contrato.md) e [18](../endpoints/whatsapp/18-observacoes-de-contrato-gateway-e-rodadas.md). Esta seção preserva o detalhe de implementação que estava na ficha.

## Regras de implementação

- Status: `NaoAtendido`, `EmAtendimento`, `AguardandoCliente`, `Resolvido`,
  `Finalizado`, `Arquivado`, `Visto` (`WhatsAppEnums.cs`); prioridade e status
  do número são enums persistidos; SLA em minutos úteis (`SlaCalculo`).
- `Mensagem.Tipo` é **string livre** (sem enum, sem schema): texto, `image`,
  `audio` e — Fases 6/7 do plano 2026-10 — `video` (mp4/webm), `gif`, `contact`.
- Telefone: a chave canônica (`NormalizarTelefone`) remove o nono dígito de
  propósito; o **cartão/envio** usa `NormalizarParaCartao` (DDI 55 com o 9
  **preservado**). A exibição usa `FormatarExibicao` (13 dígitos, com o 9) e,
  nos DTOs de contato/atendimento, `FormatarContato`: em `@lid`/grupo só o
  `TelefoneResolvido` vale; em contato privado a prioridade é
  `TelefoneResolvido ?? Numero ?? TelefoneNormalizado`.
- **`@lid` e resolução sob demanda:** `RegistrarEntradaAsync` grava em
  `WAT_Numero` o telefone de **cartão** (com o 9) do contato privado; `@lid` e
  `@g.us` **nunca** viram número; `TelefoneResolvido` recebe
  `NormalizarParaCartao`. O `ContatoTelefoneResolver`
  (`IContatoTelefoneResolver`, chamado por `ContatoVinculoService.ObterAsync`)
  consulta `GET sessions/{sessao}/contacts/{chatId}/phone`, grava em
  `WAT_TelefoneResolvido` e **não quebra a ficha** sem sessão ou com falha do
  gateway. O cache é o próprio campo; o resolvido é **display**, não identidade
  — não funde contato nem entra na duplicidade. A busca de contatos
  (`ContatoUnificacaoService.ListarAsync`, paginado, e
  `ContatoAdminService.AplicarFiltros`) inclui `WAT_TelefoneResolvido`. **Evidência pré-reset:** o backfill de
  2026-10-30 resolveu **43/43** `@lid`, mas foi **apagado** pelo reset do mesmo
  dia (ver [03](./03-testes-pendencias-e-operacao.md)); a resolução volta
  contato a contato depois da reconexão.
- **Contato só nasce no vínculo:** `POST /contatos/vincular`
  (`ContatoVinculoService.VincularAsync`, ramo de criação com `contatoId =
  null`, admin) é a **única** porta que cria linha em `tbwhatsapp_contato` —
  exige `empresaId`, usa o pushName da última mensagem quando o nome vem vazio,
  aponta as conversas do chat que estavam sem contato e grava a empresa na
  mesma ação (histórico preservado); a auditoria `vincular` leva a marca
  `contato_criado`. O mesmo `VincularAsync` atende o `PATCH /contatos/{id}`
  (`contatoId` informado → atualiza com lock otimista). A origem é a aba
  **"Sem contato"** de Admin > Contatos
  (`GET /contatos/sem-contato`), que lista chats privados com `WAT_ContatoId`
  nulo, nome do pushName da última mensagem, telefone do JID e total de
  atendimentos. `RegistrarEntradaAsync` abre `Conversa` e `Atendimento` com
  `ContatoId` nulo e grava `Mensagem`, mas **não** cadastra; grupo também não
  cria contato. `SolicitarPrivadoAsync` (grupo → privado) abre/reaproveita a
  conversa pelo `ChatId` do participante sem cadastrar. Contato existente é
  reaproveitado por `ChatId` ou telefone normalizado.
- **Iniciar conversa nunca cria contato:** `IniciarConversaAsync` exige
  **exatamente um** entre `chatId` (`@c.us`) e `telefone` (normalizado por
  `NormalizarParaCartao`: DDI 55, com o 9, DDD/tamanho validados) — os dois ou
  nenhum → `400`, assim como `@g.us`/`@lid`. Número inexistente ou sessão
  desconectada → `404 { mensagem: "Número indisponível. Verifique a conexão." }`.
  A conversa é buscada por (`NumeroId`, `ChatId`) e criada com o `ContatoId` do
  contato salvo do chat (ou nulo) — **nunca** cria contato. Atendimento ativo
  (`NaoAtendido`, `EmAtendimento`, `AguardandoCliente`, `Visto`) de outro
  operador → `409 responsavel_diferente` com `responsavelNome` e
  `atendimentoId`; sem responsável → reutiliza `AssumirAsync`; do próprio
  operador → reaproveita. Atendimento terminal (Resolvido/Finalizado/Arquivado)
  → novo atendimento na mesma conversa, `EmAtendimento` e responsável = operador.
  Não envia mensagem ao gateway.
- **Conversa sem contato salvo:** o detalhe monta `ContatoContextoDto` com
  `ContatoId = 0`, `Salvo = false`, nome vindo do pushName
  (`Mensagem.AutorNome` da última recebida) e telefone derivado do `ChatId`.
  Na lista, `AtendimentoResumo.ContatoNome` cai para o `AutorNome` da última
  mensagem recebida quando não há contato. O webhook passou a gravar
  `AutorNome` também em conversa privada (antes só em grupo) como fonte desse
  nome; o balão **não** exibe autor no privado (regra existente).
- **Tipo de contato:** `TipoContatoWhatsApp` vai de **0 a 5**, com rótulo em
  `Rotulo()` (`WhatsAppEnums.cs`); **5** = `SupervisorCobranca` ("Supervisor de
  Cobrança"). O `PATCH /contatos/{id}` valida com `Enum.IsDefined`
  (`ContatoVinculoService`) — fora da faixa é `400`. O CHECK
  `CK_tbwhatsapp_contato_TipoContato` aceita **0..5** e está **aplicado em
  2026-10-30** em `dbCENTRAL_OPERACAO` e `dbBUSINESS_HML` (`192.168.2.154`),
  com a definição `([WAT_TipoContato]>=(0) AND [WAT_TipoContato]<=(5))`; o
  [`WhatsApp_Fase6_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase6_Idempotente.sql)
  **remove a CHECK antiga (0..4)** e a recria como 0..5 — é a **única** operação
  não aditiva do script, e a reaplicação é **no-op (idempotente)**.
- **Reação (Fase 8):** habilitada por configuração desde 2026-10-30
  (`appsettings.json` com `ReacoesHabilitadas=true`); o modelo de ambiente novo
  (`appsettings.sample.json`) nasce `false` e, para desligar, use
  `WhatsApp__ReacoesHabilitadas=false`. O alvo precisa de `ExternalId`; a **nota
  interna nunca reage**; o emoji vazio remove e a remoção é **lógica**
  (`RemovidaEm`). A Central grava a reação local **somente depois do 2xx** do
  gateway — falha do gateway é `502` e não deixa estado otimista.
- **Menção em grupo:** `EntradaWhatsApp.MentionedIds` é propagado pelo
  `WhatsAppWebhookService`; o `AtendimentoService.SubstituirMencoesAsync` troca
  `@<jid>` pelo **nome** do contato salvo (por `ChatId` ou
  `TelefoneNormalizado`, depois participante do grupo por `ChatId`). Sem nome,
  usa o telefone legível; `@lid`/`@g.us` sem nome ficam **crus**. O payload
  bruto continua na fila do webhook.
- **Finalizar é terminal e silencioso:** `FinalizarAsync` só aceita `Resolvido`,
  grava `Finalizado` + evento `finalizacao`, **não envia mensagem** (request
  vazio) e **não arquiva**; só o **Resolver** fala com o cliente.
- **Finalização automática arquiva:** o job de 1 min leva `Resolvido` →
  **`Arquivado`** após a janela (`FinalizacaoAutomaticaMinutos`, padrão 15;
  `<= 0` desliga), exceto grupos, via `ArquivarCondicionalAsync`; grava
  `FinalizacaoAutomatica` + `Arquivamento` (`OperadorId = null`) e publica
  `Arquivado` no SignalR.
- **FCR:** `CalcularQualidadeAsync` conta `Resolvido`, `Finalizado` e
  `Arquivado` como resolvidos.
- **Prefixo de autor:** `AplicarPrefixoAutorAsync` aplica `*NOME:* `
  (maiúsculas, sem duplicar) nas respostas (`EnviarMensagemAsync`) e no
  **encerramento do Resolver** (`EnviarEncerramentoAsync`), usando o texto do
  número ou o digitado. Nota interna e mídia não recebem prefixo.

## Contratos que não podem quebrar

- Status é `byte`; a ordem quebra a fila. Webhook público **precisa** validar
  assinatura; ação sensível grava auditoria.
- Mídia **enviada**: imagem sem SVG, áudio, `video/mp4`, `video/webm`,
  `image/gif` — **lista fechada** de MIME, ≤ **10 MB** (request 12 MB).
  Documento arbitrário é recusado; `send-document` só serve ao fallback do
  cartão. Ctrl+V de imagem vira anexo, sem envio automático.
- **Contato**: `POST /atendimentos/{id}/mensagens/contato` (`{ nome, telefone,
  empresa? }`) → `200` com `tipo:"contact"`, `temMidia:false`, **sem mídia
  persistida**; `400` validação, `404` fora do escopo, `409` de
  responsabilidade; falha do gateway = `200` com `status=Falhou`; auditoria
  `mensagem_contato`. Gateway:
  `send-contact`; **404/501** → vCard 3.0 em `send-document` (status só no log);
  **outro status não reenvia** (não duplica o cartão).
- **Criação de contato só no vínculo:** `POST /contatos/vincular`
  (`VincularContatoRequest`: `empresaId` **obrigatório**, `chatId` obrigatório,
  `nome?` ≤ 120, `tipoContato?` 0–5, `observacao?` ≤ 500) →
  `200 ContatoFichaDto`. `400` sem empresa ("Selecione a empresa para
  vincular."), com `chatId` vazio, grupo `@g.us` ou nome > 120; `403` sem
  administrador. Reaproveita contato por `ChatId`/telefone normalizado, usa o
  `AutorNome` da última mensagem quando o nome não é informado e aponta as
  conversas do chat sem contato. Auditoria `vincular`, com a marca
  `contato_criado` quando a linha nasce; o `PATCH /contatos/{id}` reusa o mesmo
  `ContatoVinculoService.VincularAsync` com lock otimista (`versao`).
  `GET /contatos/sem-contato`
  (admin) é paginado por `busca`/`pagina`/`tamanho` e devolve
  `ContatoSemVinculoDto` (`chatId`, `nome` do pushName, `telefone` do JID,
  `numeroId`, `numeroDescricao`, `ultimaMensagemEm`, `totalAtendimentos`).
  `GET /contatos` também é paginado (`busca`/`pagina`/`tamanho`, resposta
  `Paginado<ContatoResumoDto>` com `TelefoneExibicao`) e serve à conversa, ao
  composer e ao painel Contatos do Kanban. Entrada de número desconhecido e
  privado de grupo **não** criam contato.
- **Iniciar conversa:** `POST /atendimentos/iniciar` recebe
  `IniciarConversaRequest { numeroId, chatId?, telefone? }` e exige
  **exatamente um** entre `chatId` e `telefone`; os dois ou nenhum → `400`.
  `telefone` é normalizado com `NormalizarParaCartao` (DDI 55, com o 9) e o
  DDD/tamanho são validados; `chatId` precisa terminar em `@c.us`, e `@g.us`/
  `@lid` → `400`. Número inexistente ou sessão desconectada → `404
  { mensagem: "Número indisponível. Verifique a conexão." }`. Resposta
  `200 { atendimentoId, conversaId, status, criado }`; auditoria
  `iniciar_conversa` com número, chat/telefone e operador. A conversa é
  reaproveitada por (`NumeroId`, `ChatId`) e nunca cria contato nem envia
  mensagem ao gateway.
- **Foto por chat:** `GET /contatos/foto?chatId=...` (autenticado) serve a
  imagem de conversa/participante sem contato; o `chatId` é validado, o cache
  é em memória e **não há schema novo**. `404` cai para as iniciais — é o
  mesmo proxy autenticado de `GET /contatos/{id}/foto`.
- **Integrante de grupo:** `ParticipanteWhatsApp.TelefoneExibicao` (aditivo)
  sai de `FormatarExibicao` do telefone ou dos dígitos do `ChatId` (nunca de
  `@lid`), e `ListarParticipantesAsync` prioriza `Participante.Nome` → contato
  salvo do mesmo `ChatId` → telefone, persistindo o nome descoberto. O modal
  "Integrantes" mostra avatar e nome ou telefone, nunca "Sem nome".
- O 409 de “outro operador” (`motivo:"responsavel_diferente"`, aditivo) vale
  para texto, mídia, contato e iniciar conversa; o modal da tela depende do
  discriminador (`responsavelNome` e `atendimentoId` no corpo).
- Mídia **persistida** só sai por `GET /mensagens/{id}/midia` (blob
  autenticado; lightbox reusa o mesmo blob; nenhuma URL pública).
- `busca` de `GET /atendimentos` é server-side paginada (teto 200).
- **Telefone de exibição:** `HistoricoContato.TelefoneExibicao` vem do
  `AtendimentoService` via `TelefoneNormalizador.FormatarContato` (em `@lid`/
  grupo só `TelefoneResolvido`; em privado
  `TelefoneResolvido ?? Numero ?? TelefoneNormalizado`); é **display**, não
  identidade — `ChatId` e chave normalizada seguem sendo a identidade e o `@lid`
  resolvido não funde contato. Modal de vínculo e painel do cliente priorizam
  `telefoneExibicao`, com fallback. O `ContatoTelefoneResolver` grava o
  resolvido sob demanda (`WAT_TelefoneResolvido`) e a falha do gateway não
  impede a ficha; sem resolução, a tela mostra `—` no `@lid`. O CHECK `0..5`
  também está aplicado nos dois bancos.
- **Prefixo e menção:** o texto persistido é o mesmo enviado ao gateway —
  `*NOME:* ` na frente da resposta de texto (sem duplicar) e `@<jid>` trocado
  pelo nome salvo. O **encerramento do Resolver** também recebe o prefixo, seja
  o texto configurado no número ou o digitado. Nota interna e mídia não recebem
  prefixo; o payload bruto do webhook permanece na fila.
- **Finalizar × Resolver × automação:** `POST /atendimentos/{id}/finalizar` não
  envia nada ao cliente e não arquiva; `POST /atendimentos/{id}/resolver` envia o
  encerramento quando `enviarEncerramento=true`; a automação (janela em minutos)
  leva `Resolvido` direto para **`Arquivado`**, com `OperadorId = null`. A
  configuração da janela é `GET`/`PUT /configuracao/finalizacao-automatica`
  (admin, `0..1440`, `0` desliga).
- **Hub SignalR:** o front só aceita `whatsappHubUrl` do runtime-config quando a
  origem é a **mesma** da `apiBaseUrl` e não é localhost em host remoto; caso
  contrário deriva `{origem da api}/hubs/whatsapp`. `conectar()` aguarda o auth
  inicializar e tenta de novo.
- **Fase 8 (reações):** ligada por configuração desde 2026-10-30
  (`WhatsApp__ReacoesHabilitadas`; o `appsettings.sample.json` nasce `false`). A
  rota responde `200 AtendimentoDetalhe` e falha com `400`/`404`/`409`/`502` —
  **sem estado otimista**. `tbwhatsapp_mensagem_reacao` veio de
  [`WhatsApp_Fase9_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase9_Idempotente.sql),
  **aplicado nos dois bancos** em 2026-10-30; o preflight não bloqueia mais.
  **Evidência do gateway é informal:** `messages/react`, `GET .../reactions` e
  `message.reaction` seguem **NÃO TESTADOS**. Regras, payload mínimo do hub
  (`reacao_atualizada`) e o modo limitado (C3) em
  [Atendimentos](../endpoints/whatsapp/01-atendimentos-atendimentos.md).
