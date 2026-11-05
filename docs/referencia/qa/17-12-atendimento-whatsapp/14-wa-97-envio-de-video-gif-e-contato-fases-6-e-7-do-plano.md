---
id: 17-12-atendimento-whatsapp.14
titulo: "WA-97 — Envio de vídeo, GIF e cartão de contato (Fases 6 e 7 do plano 2026-10) (+4)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppContratoGatewayTests.cs
  - backend/Central_BackEnd.Tests/WhatsAppVCardTests.cs
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
tokens_estimados: 2160
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-97 — Envio de vídeo (mp4/webm) até 10 MB (Fase 6 do plano 2026-10)

- **Objetivo (H.7):** confirmar que o operador envia vídeo pela Central e o arquivo chega como vídeo, com tipo correto no histórico e prévia/lightbox funcionando.
- **Pré-condições:** atendimento aberto com o operador como responsável (ou vazio, para testar a atribuição); vídeo `mp4` H.264 e um `webm` < 10 MB; um `.mp4` > 10 MB; um `.pdf` à mão.
- **Passos:** 1. Clicar o botão de vídeo (`bi-camera-video`) e escolher o `mp4`; opcionalmente escrever legenda. 2. Enviar e conferir o balão e o Network (`POST .../mensagens/midia`). 3. Ampliar o vídeo no lightbox. 4. Repetir com `webm`. 5. Tentar o arquivo acima de 10 MB e depois o `.pdf`.
- **Resultado esperado:** `AtendimentoService.EnviarMidiaAsync` aceita a lista fechada `video/mp4`/`video/webm` e grava `Mensagem.Tipo = "video"` com o MIME real; o `OpenWaClient` manda para `sessions/{id}/messages/send-video` com corpo **FLAT** `{ chatId, base64, mimetype, filename, caption }`. A legenda vai como `caption`. O balão reusa `MidiaMensagemComponent` (render por MIME já existia) com player e lightbox sobre o mesmo blob autenticado. Toast **"Vídeo enviado."**. Acima de **10 MB** (`MidiaMaximaBytes`; teto do request 12 MB) e com mime fora da lista (documento, `video/quicktime`, SVG) a resposta é **`400`** com mensagem genérica; o `OpenWaClient` também recusa antes de chamar o gateway.
- **Estado atual/bloqueio:** **confirmado no teste manual de 2026-10-30** (`.130`, sessão pareada): mp4 chegou via `send-video`. D5 segue **conservador** — o teto de 10 MB é imposto pela Central (`MidiaMaximaBytes`); o limite real do gateway não foi medido (`limitado(10 MB pela Central)`). Sem schema, sem migration.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/midia`, `GET /api/v1/whatsapp/mensagens/{id}/midia`; `tbwhatsapp_mensagem` (`WAT_Tipo`, `WAT_MidiaMime`, `WAT_MidiaBase64`).
- **Status/automação:** **cenário derivado do código**; coberto por `backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs` (mp4/webm aceitos, 10 MB, documento recusado) e `backend/Central_BackEnd.Tests/WhatsAppContratoGatewayTests.cs` (corpo e rota `send-video` com handler falso). Execução manual: `mp4` OK em 2026-10-30 (`.130`).

### WA-98 — Envio de GIF animado (Fase 6 do plano 2026-10)

- **Objetivo (H.7/D2):** confirmar que GIF não vira imagem estática nem documento, e que a animação chega ao destinatário.
- **Pré-condições:** `.gif` animado < 10 MB; celular com WhatsApp para conferir o recebimento.
- **Passos:** 1. Enviar o GIF pelo botão de vídeo (`accept` inclui `image/gif`). 2. Conferir o chip do anexo: **ícone por tipo** e prévia de imagem exibida para GIF. 3. Enviar e abrir o balão no histórico. 4. Conferir no celular de destino se o cartão aparece **animado**.
- **Resultado esperado:** `Tipo = "gif"` (string livre, sem enum novo) e o **mimetype real preservado** no corpo do gateway — o `OpenWaClient` roteia `image/gif` para `send-video`, e não para `send-image`, exatamente para não rebaixá-lo a imagem estática. No balão, `image/gif` cai no caminho de **imagem** do `MidiaMensagemComponent` (prévia + lightbox). Toast **"GIF enviado."**.
- **Estado atual/bloqueio:** **D2 decidido em 2026-10-30 com evidência** — teste manual (`.130`, sessão pareada): o GIF chegou **animado** no celular via `send-video` com mimetype real `image/gif`, **sem** flag (`gifPlayback`/`isGif` não são necessários). Registro canônico: § 3.6.3 de `material-de-projeto/atendimento-whatsapp/PLANO-EVOLUCAO-WHATSAPP-2026-10.md`. Sem schema.
- **Rotas/endpoints/tabelas:** idem WA-97; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; tipo `gif` coberto em `backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs` e pela rota `send-video` em `WhatsAppContratoGatewayTests.cs`. Validação visual da animação: **manual, OK em 2026-10-30**.

### WA-99 — Cartão de contato pelo caminho A (`send-contact`) (Fase 7 do plano 2026-10)

- **Objetivo (H.8):** enviar um contato estruturado (nome, telefone, empresa opcional) pela Central e vê-lo como **card** no balão do histórico.
- **Pré-condições:** atendimento aberto na fila; mini-form de contato visível no composer (`bi-person-lines-fill`); celular de destino.
- **Passos:** 1. Abrir o mini-form e digitar nome, telefone (a máscara aceita só dígitos) e empresa. 2. Confirmar: nasce o chip "nome · telefone". 3. Enviar. 4. Conferir o balão, o Network e o recebimento no WhatsApp do cliente. 5. Confirmar que o balão **não** dispara `GET /mensagens/{id}/midia`.
- **Resultado esperado:** `POST /atendimentos/{id}/mensagens/contato` com `{ nome, telefone, empresa? }`; o gateway recebe `send-contact` com `{ chatId, contactName, contactNumber }` e o número em dígitos **E.164 sem `+`**. A mensagem é gravada com `Tipo = "contact"`, `Texto = "Contato: {nome} ({telefone formatado})"` e **sem mídia persistida** (`temMidia: false`). Resposta `200 AtendimentoDetalhe`; auditoria `acao "mensagem_contato"`. O card do balão (`whatsapp-message.component.ts`, `ehContato`) lê o rótulo com parsing defensivo e cai no texto cru se o formato mudar. Toast **"Contato enviado."**. **Nada de enum ou coluna nova.**
- **Estado atual/bloqueio:** o número do cartão é o de **discagem** — `TelefoneNormalizador.NormalizarParaCartao` garante DDI 55 e **preserva o nono dígito**; a chave canônica do cadastro (`NormalizarTelefone`) remove o 9 e **não pode** ir no cartão, sob pena de entregar número que não disca. A existência da rota `send-contact` no OpenWA v0.25.0 é **presunção** (Fase 0 não executada) — validar manualmente.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/contato`; `tbwhatsapp_mensagem`, `tbwhatsapp_auditoria`; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; coberto por `backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs` (contato feliz e com empresa) e `backend/Central_BackEnd.Tests/WhatsAppContratoGatewayTests.cs` (payload `send-contact`). Validação visual do card no WhatsApp: **manual, pendente**.

### WA-100 — Fallback do cartão: vCard `.vcf` anexado quando 404/501 (Fase 7)

- **Objetivo (D2):** confirmar que, sem a rota `send-contact` no gateway, o cartão ainda chega como arquivo `.vcf` válido — e que **nenhum outro erro gera reenvio**.
- **Pré-condições:** ambiente com gateway em que a rota `send-contact` não exista (ou o falso de teste devolvendo 404/501); outro devolvendo 500.
- **Passos:** 1. Enviar o cartão com a rota ausente no gateway. 2. Conferir nos logs do backend o aviso do fallback e a chamada seguinte. 3. Verificar no WhatsApp o anexo `.vcf` e abri-lo. 4. Simular **500** na primeira chamada e contar quantas requisições saíram. 5. Conferir o conteúdo do cartão com nome contendo vírgula, ponto e vírgula, contra-barra e quebra de linha.
- **Resultado esperado:** em **404/501** o cliente chama `send-document` com `data:text/vcard;base64,`, `mimetype: "text/vcard"` e filename em slug `.vcf` (`WhatsAppVCard.NomeArquivo`), gerando vCard **3.0** com `ORG` só quando a empresa foi informada. O status 404/501 fica **somente no log** — a resposta à tela continua `200` com o card gravado. Em **qualquer outro status não há reenvio** (reenviar em 500 duplicaria o cartão). A sanitização remove CR/LF e escapa `\`, `,` e `;`, então o campo digitado não injeta linhas novas no cartão.
- **Estado atual/bloqueio:** caminho B é o plano explícito para a engine sem `send-contact`; continua sendo **presunção a validar** contra o OpenWA v0.25.0. Sem schema.
- **Rotas/endpoints/tabelas:** rota interna do gateway `sessions/{id}/messages/send-document`; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; coberto por `backend/Central_BackEnd.Tests/WhatsAppContratoGatewayTests.cs` (fallback 404 e 500 **sem** fallback) e `backend/Central_BackEnd.Tests/WhatsAppVCardTests.cs` (geração e sanitização). Abrir o `.vcf` no celular: **manual, pendente**.
