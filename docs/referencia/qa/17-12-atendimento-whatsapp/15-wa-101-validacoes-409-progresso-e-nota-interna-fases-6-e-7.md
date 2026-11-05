---
id: 17-12-atendimento-whatsapp.15
titulo: "WA-101 — Validações, 409, progresso e nota interna no envio (Fases 6 e 7) (+4)"
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
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/components/whatsapp-composer/whatsapp-composer.component.spec.ts
tokens_estimados: 1740
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-101 — Validações do cartão: telefone inválido e nome vazio respondem 400 (Fase 7)

- **Objetivo (H.8):** nenhum dado absurdo chega ao gateway nem vira cartão inútil para quem recebe.
- **Pré-condições:** acesso à API com token de atendente; composer aberto.
- **Passos:** 1. Enviar com nome vazio/só espaços. 2. Enviar telefone com letras, com menos de 12 dígitos já com DDI ou com mais de 15. 3. Enviar nome com 121 caracteres e empresa com 121. 4. Digitar letras no campo de telefone da tela.
- **Resultado esperado:** `ArgumentException` → **`400`** com mensagem genérica ("Informe o nome do contato.", "Informe um telefone válido para o contato.", máximos de 120 caracteres); nada é gravado e nada vai ao gateway. O campo da tela **filtra não dígitos** e limita o comprimento. Empresa é opcional.
- **Estado atual/bloqueio:** o intervalo 12–15 dígitos é do **número de discagem** já com DDI 55 — fora dele o que foi digitado é texto solto, não telefone.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/contato`; nenhuma tabela.
- **Status/automação:** **cenário derivado do código**; coberto por `backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs` (telefone inválido, nome vazio) e pelos specs do composer.

### WA-102 — 409 de responsabilidade também no envio de mídia e de contato (Fases 6 e 7)

- **Objetivo:** o modal de responsabilidade vale para **todo** envio da Central, não só para texto.
- **Pré-condições:** card já atribuído a outro operador; segundo caso com o **modal** aberto (assumir e reenviar).
- **Passos:** 1. Tentar enviar mídia de um card de outro operador. 2. Tentar o mesmo com o cartão de contato. 3. Confirmar o modal "Atendimento de outro operador" → transferir → reenvio.
- **Resultado esperado:** `409 { motivo: "responsavel_diferente" }` — os dois envios reusam `AtendimentoService.EnviarPelaCentralAsync` (atribuição na primeira resposta, commit **antes** do gateway). No reenvio pós-modal a mesma payload parte **uma** vez (sem duplicar anexo nem cartão). Falha do gateway **não** reverte atribuição/retomada: a mensagem fica `StatusMensagemWhatsApp.Falhou` e a resposta continua `200` (padrão dos demais envios).
- **Estado atual/bloqueio:** nenhum contrato novo; o discriminador aditivo do 409 já existia para texto.
- **Rotas/endpoints/tabelas:** `POST /atendimentos/{id}/mensagens/midia`, `POST /atendimentos/{id}/mensagens/contato`, `POST /atendimentos/{id}/transferir`; `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`.
- **Status/automação:** **cenário derivado do código**; coberto por `backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs` (409 e falha do gateway no envio de contato).

### WA-103 — Barra de progresso do upload e reenvio silencioso (Fases 6 e 7)

- **Objetivo (H.7):** dar retorno visual durante o upload de até 10 MB sem quebrar quem chama `enviarMidia` com três argumentos.
- **Pré-condições:** throttle lento no DevTools e um vídeo grande.
- **Passos:** 1. Anexar um vídeo e enviar: acompanhar `.wa-composer__progresso` até sumir. 2. Repetir com legenda. 3. Enviar texto/contato e conferir que **não** há barra. 4. Disparar o reenvio pelo modal de responsabilidade e observar o progresso.
- **Resultado esperado:** `enviarMidia(id, arquivo, legenda, progresso?)` só usa `observe: 'events'` e `reportProgress` **quando o callback é passado** — o contrato antigo permanece intacto para as demais chamadas e specs. A barra expõe `aria-valuenow` e some no fim. O **reenvio pós-modal é silencioso** (sem barra), porque o usuário já viu o envio original.
- **Estado atual/bloqueio:** frontend puro; nenhum endpoint muda. Não há progresso de **download** — o blob continua sob demanda.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/midia`; nenhuma tabela.
- **Status/automação:** **cenário derivado do código**; coberto pelos specs do composer e da página. Percepção da barra em upload real: **manual**.

### WA-104 — Nota interna sem anexo e sem contato (Fases 6 e 7)

- **Objetivo:** nada que não seja texto escapa para o WhatsApp quando o destino é interno.
- **Pré-condições:** composer com anexo (imagem, vídeo ou GIF) **e**, em outro teste, contato pendente confirmado.
- **Passos:** 1. Anexar mídia e trocar o destino para "Nota interna". 2. Abrir o mini-form de contato em modo nota. 3. Confirmar o contato com um anexo já pendente e, depois, escolher anexo com contato pendente. 4. Salvar a nota e abrir o card de outro operador.
- **Resultado esperado:** trocar para nota interna **mantém o texto e descarta** anexo/contato com aviso local ("Contato removido: nota interna não envia contato."); nota interna nunca grava mídia nem cartão. Anexo e contato são **mutuamente exclusivos**: confirmar o contato remove o anexo ("contato e mídia não se misturam no mesmo envio") e escolher anexo limpa o contato. O seletor de destino continua sendo o único ponto que decide para onde vai o envio.
- **Estado atual/bloqueio:** sem mudança de contrato; `POST .../notas` continua gravando só texto (direção 2).
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/atendimentos/{id}/notas`; `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário derivado do código**; coberto por `frontend/src/app/features/whatsapp/components/whatsapp-composer/whatsapp-composer.component.spec.ts` e pelo backend `WhatsAppAtendimentoTests.cs`.

> **Evidência e pendência comuns às Fases 6 e 7:** o teste manual de **2026-10-30** (estação `.130`, sessão pareada) **confirmou GIF animado via `send-video` sem flag (D2 decidido), vídeo mp4 e o envio de contato** — registro canônico em `material-de-projeto/atendimento-whatsapp/PLANO-EVOLUCAO-WHATSAPP-2026-10.md` (§ 3.6.3 e § 4.1). D5 segue **conservador**: teto de 10 MB imposto pela Central (`MidiaMaximaBytes`); o limite real do gateway não foi medido. **Presunções**: caminho A/B do contato (o envio chegou, o operador não distinguiu `send-contact` nativo vs `.vcf` via `send-document`) e a versão exata do container (não reconfirmada por `/health`; vale a imagem fixada `ghcr.io/rmyndharis/openwa:0.25.0`). **A Fase 8 (reações) está habilitada por configuração desde 2026-10-30** (`appsettings.json` com `ReacoesHabilitadas=true`; o modelo de ambiente novo nasce `false`) e o script `WhatsApp_Fase9_Idempotente.sql` (`tbwhatsapp_mensagem_reacao`) **foi aplicado nos dois bancos de homologação**, mas a medição do gateway continua **adiada**: só há evidência informal, sem status HTTP nem payload. Cenários em [WA-105](./16-wa-105-reacoes-atras-da-flag-fase-8.md).
