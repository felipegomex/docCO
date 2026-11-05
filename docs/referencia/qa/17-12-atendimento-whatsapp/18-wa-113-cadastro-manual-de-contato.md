---
id: 17-12-atendimento-whatsapp.18
titulo: "WA-113 — Contato no vínculo: conversa e Admin (rodada 2026-11-03) (+4)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias, contato, vinculo, pushname]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppContatoTests.cs
  - backend/Central_BackEnd.Tests/WhatsAppGrupoAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/components/cliente-context-panel/cliente-context-panel.component.spec.ts
tokens_estimados: 1500
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-113 — Chat sem contato aparece na aba "Sem contato" (rodada 2026-11-03)

- **Objetivo:** localizar os chats privados que ainda não têm contato e usá-los como origem do vínculo.
- **Pré-condições:** número conectado; mensagens recebidas de números que não existam em `tbwhatsapp_contato` (nem por `ChatId`, nem por telefone normalizado); usuário administrador.
- **Passos:** 1. Enviar mensagem de um número desconhecido. 2. Abrir Admin > Contatos → aba **"Sem contato"**. 3. Buscar pelo nome/telefone e paginar. 4. Conferir `tbwhatsapp_conversa` e `tbwhatsapp_contato`.
- **Resultado esperado:** `GET /contatos/sem-contato` devolve `200 Paginado<ContatoSemVinculoDto>` só com chats **privados** de `WAT_ContatoId` nulo, trazendo o nome do pushName da última mensagem, o telefone do JID e o total de atendimentos. Receber a mensagem **não** cria linha em `tbwhatsapp_contato`.
- **Estado atual/bloqueio:** `ContatoAdminService.ListarSemContatoAsync`; aba em `pages/admin-contatos`.
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/contatos/sem-contato`; `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_contato`.
- **Status/automação:** **cenário derivado do código**; suíte `WhatsApp*` (service/controller) e QA manual.

### WA-114 — Entrada de número desconhecido não cria contato (rodada 2026-11-03)

- **Objetivo:** receber mensagem de um número desconhecido sem cadastrá-lo automaticamente e ainda exibir o pushName.
- **Pré-condições:** número conectado; remetente que não exista em `tbwhatsapp_contato`; para grupo, um `@g.us` sem contato.
- **Passos:** 1. Enviar mensagem de um número desconhecido. 2. Abrir o atendimento e o painel do cliente. 3. Repetir com mensagem em grupo. 4. Conferir `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento` e `tbwhatsapp_mensagem`.
- **Resultado esperado:** `RegistrarEntradaAsync` abre `Conversa` (`ContatoId` nulo) e `Atendimento` e grava a `Mensagem`, mas **não** cria contato — nem para grupo. O detalhe monta `ContatoContextoDto` com `ContatoId = 0`, `Salvo = false`, nome do pushName (`AutorNome` da última recebida) e telefone derivado do `ChatId`; a lista usa o `AutorNome` em `ContatoNome`. O balão continua **sem** exibir autor no privado.
- **Estado atual/bloqueio:** `AtendimentoService.RegistrarEntradaAsync`; `WhatsAppWebhookService` grava `AutorNome` também em conversa privada como fonte do pushName.
- **Rotas/endpoints/tabelas:** webhook `POST /api/v1/whatsapp/webhook/openwa`; `GET /api/v1/whatsapp/atendimentos/{id}`; `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_contato`.
- **Status/automação:** **cenário derivado do código**; coberto por `Entrada_de_numero_desconhecido_nao_cadastra_contato` em `WhatsAppContatoTests.cs`.

### WA-115 — Vincular empresa cria o contato e assume o histórico do chat (rodada 2026-11-03)

- **Objetivo:** criar o contato pelo vínculo — no **painel da conversa** ou na aba "Sem contato" —, vinculando a empresa e assumindo as conversas do chat sem perder histórico.
- **Pré-condições:** chat privado sem contato (`WAT_ContatoId` nulo); pelo menos uma mensagem recebida com pushName; operador com acesso ao canal (a aba "Sem contato" exige administrador); opcionalmente, um contato **pendente** com o mesmo telefone normalizado.
- **Passos:** 1. Abrir o atendimento de um chat sem contato e clicar em **"Vincular empresa"** no bloco **"Contato ainda não vinculado"**. 2. Conferir o **Nome** pré-preenchido com o pushName, selecionar a empresa no typeahead e confirmar. 3. Conferir a ficha e as conversas do chat; o bloco deve virar **"Cliente identificado"**. 4. Repetir pelo caminho **Admin > Contatos → "Sem contato" → Vincular empresa** e com um chat cujo telefone já tenha contato pendente.
- **Resultado esperado:** `POST /contatos/vincular` responde `200 ContatoFichaDto` e **cria ou reaproveita** o contato (por `ChatId` ou telefone normalizado) **sem sobrescrever o nome** existente, grava a empresa na mesma ação e aponta as conversas do chat sem contato para ele. O modal exige o nome; sem nome no backend, usa o `AutorNome` da última mensagem. Conversas já vinculadas a outro contato são preservadas. Auditoria `vincular` com `contato_criado` quando a linha nasce.
- **Estado atual/bloqueio:** `ContatoVinculoService.CriarEVincularAsync`; `cliente-context-panel` + `vincular-empresa-modal` (conversa) e `pages/admin-contatos` (aba "Sem contato").
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/contatos/vincular` (`VincularContatoRequest`); `/whatsapp/atendimento/:id`; `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_auditoria`.
- **Status/automação:** **cenário derivado do código**; suíte `WhatsApp*` (service/controller) e QA manual.

### WA-116 — Grupo e empresa ausente são recusados no vínculo (rodada 2026-11-03)

- **Objetivo:** impedir que grupo vire contato e que o vínculo seja criado sem empresa.
- **Pré-condições:** chat de grupo (`ChatId` terminando em `@g.us`) e um administrador autenticado.
- **Passos:** 1. Enviar `POST /contatos/vincular` com o `chatId` do grupo e `empresaId` válido. 2. Repetir sem `empresaId`, com `chatId` vazio e com nome acima de 120 caracteres. 3. Conferir `tbwhatsapp_contato`.
- **Resultado esperado:** `400` "Grupo não é contato: escolha uma conversa privada." para o grupo; `400` "Selecione a empresa para vincular." sem empresa; `400` "Informe o chat da conversa." para `chatId` vazio; `400` para nome > 120; `403` sem acesso ao canal (`perm:aba.suporte.whatsapp:Ler`). **Nenhuma** linha é criada ou alterada em `tbwhatsapp_contato`, a auditoria não registra a ação e o grupo **não** aparece em `/contatos/sem-contato`.
- **Estado atual/bloqueio:** validações em `ContatoVinculoService.CriarEVincularAsync`; o controller devolve `BadRequest` com `{ mensagem }` e `403` para `ForbiddenException`.
- **Rotas/endpoints/tabelas:** `POST /api/v1/whatsapp/contatos/vincular`; `tbwhatsapp_contato`.
- **Status/automação:** **cenário derivado do código**; suíte `WhatsApp*` (service/controller) e QA manual.

### WA-117 — Integrantes do grupo com nome e foto (rodada 2026-11-03)

- **Objetivo:** o modal "Integrantes" nunca mostrar "Sem nome" e exibir avatar quando houver foto.
- **Pré-condições:** grupo com três tipos de participante: com nome no gateway, sem nome mas com contato salvo pelo mesmo `ChatId`, e sem nome nem contato; número conectado.
- **Passos:** 1. Abrir o atendimento do grupo e o modal **"Integrantes"**. 2. Conferir nome/telefone e avatar de cada participante. 3. Vincular um contato a um dos participantes e reabrir o modal. 4. Repetir com a sessão do gateway parada.
- **Resultado esperado:** `ListarParticipantesAsync` compõe o nome na prioridade `Participante.Nome` (gateway/pushName) → contato salvo pelo mesmo `ChatId` → telefone formatado e **persiste** o nome descoberto; `ParticipanteWhatsApp.TelefoneExibicao` (aditivo) vem do telefone ou dos dígitos do `ChatId` (nunca de `@lid`). O modal mostra avatar (`ContatoAvatarComponent` com `chatId`, foto por `GET /contatos/foto?chatId=`) e `nomeOuTelefone` — **nunca** "Sem nome". Falha do gateway mantém o cadastro local.
- **Estado atual/bloqueio:** `WhatsAppGrupoService.ListarParticipantesAsync`, `WhatsAppFotoContatoService`; `grupo-integrantes-modal`, `ContatoAvatarComponent`, `whatsapp.utils.ts`.
- **Rotas/endpoints/tabelas:** `GET /api/v1/whatsapp/grupos/{id}/participantes`, `GET /api/v1/whatsapp/contatos/foto?chatId=`; `tbwhatsapp_participante`, `tbwhatsapp_contato`.
- **Status/automação:** **cenário derivado do código**; `WhatsAppGrupoAtendimentoTests.cs`, `whatsapp.utils.spec.ts` e specs do modal.
