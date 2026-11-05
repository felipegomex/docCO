---
id: 17-12-atendimento-whatsapp.12
titulo: "WA-91 — Autoria no balão, modal de responsabilidade, negrito e Ctrl+V de imagem (Fases 1–4 do plano 2026-10) (+3)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/components/whatsapp-conversation/whatsapp-message.component.spec.ts
  - frontend/src/app/features/whatsapp/pages/atendimento/atendimento-modal-responsabilidade.component.spec.ts
  - frontend/src/app/features/whatsapp/components/whatsapp-composer/whatsapp-composer.component.spec.ts
tokens_estimados: 1500
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-91 — Autoria no balão enviado (Fase 1 do plano 2026-10)

- **Objetivo:** confirmar que o balão mostra quem enviou cada mensagem — inclusive as enviadas pela Central — com o nome do operador ou o rótulo fixo quando não há operador registrado.
- **Pré-condições:** conversa com enviada **com** `operadorId`/`operadorNome` (envio pela Central), enviada **sem** `OperadorId` (disparo direto no celular/automação — `message.sent` sem registro de autor), recebida em **grupo**, recebida em privado e nota interna; histórico completo do cliente com enviadas antigas.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id`. 2. Conferir o balão de uma enviada por operador. 3. Conferir o balão de um disparo direto no celular. 4. Abrir conversa de **grupo** e conferir as recebidas. 5. Conferir recebida no privado e a nota interna. 6. Abrir o histórico completo do cliente e conferir a autoria das enviadas.
- **Resultado esperado:** `mostrarAutor()` cobre nota, **enviada** e recebida em grupo. Na enviada com `operadorId`, o nome (`operadorNome`, fallback "Atendente") aparece em `<strong>`; na enviada sem `OperadorId`, o rótulo fixo **"WhatsApp da empresa"** (decisão registrada no plano § 5). A cor por `corDoNome` e o autor colorido continuam **só na recebida em grupo**; recebida em privado não mostra autor no balão; nota mantém "Nota interna". O histórico também projeta a autoria (`HistoricoMensagem` agora espelha `operadorId`, campo já projetado pelo backend).
- **Estado atual/bloqueio:** frontend puro (Fases 1 e 3 tocam só o balão); sem mudança de backend, contrato ou banco. Enviada com `operadorId` e nome ausente cai em "Atendente".
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; nenhum endpoint novo; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; coberto por `frontend/src/app/features/whatsapp/components/whatsapp-conversation/whatsapp-message.component.spec.ts` (rótulo sem operador, `operadorNome` em `<strong>`, fallback "Atendente").
### WA-92 — Modal de responsabilidade: 409 → transferir → reenvio único (Fase 2 do plano 2026-10)

- **Objetivo:** confirmar que responder em atendimento de outro operador abre o modal de responsabilidade, que "Sim, assumir" transfere e reenvia a mesma mensagem **uma única vez**, e que "Não, voltar" nunca perde o rascunho.
- **Pré-condições:** dois operadores com sessão em navegadores diferentes (A = responsável atual; B = quem tenta responder); atendimento **Em atendimento** com A como responsável; backend novo publicado no ambiente de teste.
- **Passos:** 1. Com A, responder normalmente (caminho feliz, sem modal). 2. Com B, digitar e enviar: conferir o `409` no Network e o modal. 3. Clicar "Não, voltar" e conferir o rascunho no campo. 4. Reenviar e clicar "Sim, assumir": conferir a ordem `POST /transferir` → `POST /mensagens` e **uma única entrega** no WhatsApp do cliente. 5. Repetir com anexo (imagem): a mesma payload (anexo + legenda) deve ser reenviada. 6. Simular corrida: entre o 409 e o "Sim, assumir", transferir o card para um terceiro operador — o `transferir` devolve 409 e o modal segue aberto. 7. Operador sem permissão: conferir o 403. 8. Derrubar a rede antes do "Sim, assumir".
- **Resultado esperado:** o corpo do 409 de "outro operador" traz `motivo=responsavel_diferente` (demais 409, como "Resolvido", vêm com `motivo` nulo e mantêm o toast antigo). O modal "Atendimento de outro operador" (`.wa-assumir`) mostra o responsável atual (e desde quando) e quem está tentando responder. "Sim, assumir" chama `POST /atendimentos/{id}/transferir` com o operador logado e, no sucesso, reenvia a **mesma payload** uma única vez (trava `assumindo`); o composer só limpa no 2xx. "Não, voltar" fecha sem enviar e preserva o rascunho. Corrida no `transferir` → toast "Outro operador mexeu neste atendimento. Confirme com o responsável atual." e o modal permanece; 403 → "Você não tem permissão para assumir este atendimento." e **não reenvia**; rede caída (status 0) → o modal fecha com o rascunho intacto no campo.
- **Estado atual/bloqueio:** única mudança de contrato do ciclo, **aditiva** (`motivo` no corpo 409); sem banco/migration. A garantia final contra corrida é do servidor (UPDLOCK + UPDATE condicional em `EnviarPelaCentralAsync`), não da tela.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/mensagens`, `POST /api/v1/whatsapp/atendimentos/{id}/mensagens/midia`, `POST /api/v1/whatsapp/atendimentos/{id}/transferir`; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; backend em `backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs` (`Resposta_de_quem_nao_e_responsavel_vem_com_motivo_responsavel_diferente` e reforço do teste de corrida) e frontend em `frontend/src/app/features/whatsapp/pages/atendimento/atendimento-modal-responsabilidade.component.spec.ts` (409→modal, transferir + 1 reenvio, voltar preserva payload, 403/409 não reenviam).
### WA-93 — Negrito `*...*` no composer e no balão (Fase 3 do plano 2026-10)

- **Objetivo:** o operador formata negrito ao digitar e vê a prévia no balão, mas o texto viaja **puro** — quem renderiza o negrito é o WhatsApp do destinatário.
- **Pré-condições:** composer em modo mensagem; texto com seleção e texto sem seleção; um balão com par `*...*`, outro com asterisco sem par e outro com vários pares.
- **Passos:** 1. Selecionar uma palavra e clicar no botão **B**. 2. Sem seleção, pressionar **Ctrl+B**. 3. Enviar e inspecionar o payload no Network. 4. Conferir o balão enviado e o recebida com `*par*`. 5. Digitar `*aberto sem fechar` e conferir. 6. Conferir no celular/WhatsApp Web do destinatário.
- **Resultado esperado:** com seleção, o texto é envolvido em `*...*`; sem seleção, o par é inserido com o cursor no meio (`aplicarNegrito()`). O campo e o payload seguem **texto puro com os asteriscos** — nada de HTML ou markdown. No balão, `segmentosNegrito()` fatia o texto e o template renderiza `<strong>` via `*ngFor` (escape-first, sem `innerHTML`): par vira negrito visual, asterisco sem par fica **literal**, múltiplos pares funcionam. Sem mudança de backend.
- **Estado atual/bloqueio:** frontend puro. O negrito visual no balão é **prévia** — a formatação nativa exibida ao cliente é decidida pelo WhatsApp do destinatário.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; nenhum endpoint novo; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; coberto por `frontend/src/app/features/whatsapp/components/whatsapp-composer/whatsapp-composer.component.spec.ts` (com/sem seleção) e `frontend/src/app/features/whatsapp/components/whatsapp-conversation/whatsapp-message.component.spec.ts` (par vira `<strong>`, resto fica literal).
### WA-94 — Ctrl+V de imagem com prévia no composer (Fase 4 do plano 2026-10)

- **Objetivo:** print/imagem na área de transferência vira anexo com miniatura no chip, sem enviar sozinho; colar texto continua texto.
- **Pré-condições:** composer em modo mensagem; imagem (screenshot ou arquivo) no clipboard; um PNG **> 10 MB**; um SVG; texto no clipboard; composer em modo nota interna.
- **Passos:** 1. Ctrl+V com imagem. 2. Conferir o chip com a miniatura. 3. Ctrl+V com texto. 4. Ctrl+V imagem em modo **nota interna**. 5. Ctrl+V imagem > 10 MB e SVG. 6. Trocar/remover o anexo e conferir a revogação da prévia. 7. Enviar pelo botão normalmente.
- **Resultado esperado:** `(paste)` → `aoColar()`: havendo **arquivo de imagem** no clipboard, o evento é `preventDefault()`-ado e a imagem vira anexo via `aceitarArquivo` — as mesmas validações do seletor (**10 MB**, **sem SVG**, com erro local quando barrada). O chip ganha `<img class="wa-composer__previa">` gerada por `URL.createObjectURL` e revogada ao trocar/remover (`urlPrevia`). Colar **texto** não é interceptado; nota interna **ignora** imagem colada; **não há envio automático** — quem dispara é o fluxo normal (`POST /mensagens/midia`, com o texto do campo como legenda). Desde a **Fase 6** a mídia enviada deixou de ser só imagem/áudio: entram também vídeo `mp4`/`webm` e GIF, no mesmo teto de 10 MB.
- **Estado atual/bloqueio:** frontend puro; sem mudança de backend/contrato. O clipboard só entrega arquivo de imagem quando a aplicação de origem o fornece (screenshot do Print Screen funciona; cópia de alguns visualizadores pode entregar só caminho).
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; nenhum endpoint novo; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; coberto por `frontend/src/app/features/whatsapp/components/whatsapp-composer/whatsapp-composer.component.spec.ts` (imagem vira anexo com prévia, > 10 MB barrado, nota interna ignora).
