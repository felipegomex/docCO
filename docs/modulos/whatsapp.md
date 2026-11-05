---
id: mod.whatsapp
titulo: "Atendimento WhatsApp"
tipo: modulo
area: atendimento
sistema: central
tags: [whatsapp, atendimento, kanban, numero, webhook, sla, metrica, etiqueta, reacao]
dono: suporte
status: ativo
revisado_em: "2026-11-03"
fonte: [backend/Central_BackEnd/Controllers/WhatsApp, backend/Central_BackEnd/Services/WhatsApp, backend/Central_BackEnd/Models/WhatsApp, frontend/src/app/features/whatsapp]
testes: [backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs, backend/Central_BackEnd.Tests/WhatsAppVCardTests.cs, backend/Central_BackEnd.Tests/WhatsAppContratoGatewayTests.cs, backend/Central_BackEnd.Tests/WhatsAppContatoTests.cs, backend/Central_BackEnd.Tests/WhatsAppContatoVinculoTests.cs, backend/Central_BackEnd.Tests/WhatsAppGrupoAtendimentoTests.cs]
tokens_estimados: 2050
---

# Atendimento WhatsApp

> Ficha. Detalhe de implementação, mapeamento de edição, testes e operação em
> [Atendimento WhatsApp — implementação, contratos e operação](../referencia/whatsapp/INDEX.md).

## 1. Quando usar este card

Fila e Kanban, painel da conversa, ciclo de vida, números/QR, métricas/SLA,
webhook OpenWA, etiquetas, respostas rápidas, retenção, contatos e envio de
mídia/cartão.

## 2. Propósito e escopo

Recebe eventos do gateway, mantém a fila por status, entrega a conversa ao
operador e mede o atendimento. Não cobre Chat/RAG (`chat-conhecimento.md`),
Kanban de tarefas (`kanban.md`) nem a operação do gateway fora da Central.

## 3. Rotas do frontend

`frontend/src/app/features/whatsapp/whatsapp.routes.ts`; rota pai exige
`aba.suporte.whatsapp`. Rotas: `/whatsapp`, `/atendimento/:id`,
`/minhas-conversas[/:id]`, `/numeros`, `/metricas`, `/admin`,
`/admin/contatos`, `/etiquetas`, `/sla`, `/retencao`, `/respostas-rapidas`,
`/auditoria`, `/configuracao`, `/ia-configuracao`.

## 4. Endpoints do backend

Base `api/v1/whatsapp`; controllers em
`backend/Central_BackEnd/Controllers/WhatsApp`; contagens e contrato rota a rota
em [Endpoints WhatsApp](../referencia/endpoints/whatsapp/INDEX.md).

| Rotas | Autorização | Controller |
|---|---|---|
| `/atendimentos` + ciclo de vida, `iniciar`, `reacao` e envios | `perm:aba.suporte.whatsapp:Ler`; arquivar só admin | `WhatsAppAtendimentosController.cs` |
| `/numeros`, `/contatos`, `/etiquetas`, `/sla`, `/retencao`, `/configuracao`, `/monitor`, `/metricas`, `/saude`, `/auditoria`, `/grupos`, `/respostas-rapidas` | escrita só admin (`.numeros`/`.metricas`/`.respostas-rapidas`) | controllers homônimos |
| `/contatos/sem-contato`; `POST /contatos/vincular` | aba Sem contato: `Administrador`; criar/vincular contato pendente: `perm:aba.suporte.whatsapp:Ler` | `WhatsAppContatosController.cs` |
| `/webhook/openwa` | anônimo + HMAC | `WhatsAppWebhookController.cs` |

`GET /contatos` aceita **`empresaId?`** opcional (a empresa do vínculo com o ERP), combinado em **AND** com `busca`; é o filtro do painel **Contatos** do Kanban, cujo typeahead usa `GET /contatos/empresas` (mínimo 2 caracteres).

## 5. Tabelas e entidades

`tbwhatsapp_*` (**22 tabelas**), sem migration EF; entidades em
`backend/Central_BackEnd/Models/WhatsApp` (`Atendimento`, `Conversa`,
`Mensagem`, `Contato`, `Numero`, `Grupo`, `Etiqueta`, `SlaConfig`,
`WebhookFila`, `Evento`, `Auditoria`, `MensagemReacao`). A 22ª
(`tbwhatsapp_mensagem_reacao`) nasce de `WhatsApp_Fase9_Idempotente.sql`,
aplicado em 2026-10-30; reset do mesmo dia em
[03](../referencia/whatsapp/03-testes-pendencias-e-operacao.md).

## 6. Serviços e lógica

Backend em `backend/Central_BackEnd/Services/WhatsApp` (`AtendimentoService` +
`CicloVida`, `WhatsAppFinalizacaoAutomaticaService`, `WhatsAppMetricasService`,
`OpenWaClient`, `ContatoTelefoneResolver`, `WhatsAppWebhookService`,
`TelefoneNormalizador`, `WhatsAppVCard`, `ContatoVinculoService`,
`WhatsAppGrupoService`); frontend em `frontend/src/app/features/whatsapp`
(`whatsapp.service.ts`, `WhatsAppRealtimeService`, `ContatoAvatarComponent`).
Responsabilidades em [01](../referencia/whatsapp/01-servicos-e-mapeamento.md).

## 7. Regras de negócio

- Status `NaoAtendido`, `EmAtendimento`, `AguardandoCliente`, `Resolvido`,
  `Finalizado`, `Arquivado`, `Visto` (`WhatsAppEnums.cs`); SLA em minutos úteis
  (`SlaCalculo`).
- Ciclo de vida (atribuição, assumir/transferir/devolver, resolver, reabrir,
  visto, métricas) em
  [16](../referencia/regras-negocio/16-modulo-de-atendimento-whatsapp-atribuicao-ciclo-de-vida-e-ag.md);
  protocolo, foto e telefone em
  [25](../referencia/regras-negocio/25-atendimento-whatsapp-protocolo-foto-telefone-e-schema.md).
- **Contato nasce no vínculo (conversa ou Admin):** `POST /contatos/vincular`
  (`empresaId` + `chatId`; nome obrigatório no modal, pré-preenchido com o
  pushName) é a única porta de criação — pelo botão **Vincular empresa** do
  painel da conversa ou pela aba "Sem contato" do Admin. Reaproveita contato
  pendente sem renomear; chat já identificado → `409
  CONFLITO_CONTATO_IDENTIFICADO`; `chatId` sem conversa existente → `400`.
  Entrada de número desconhecido e privado de grupo não cadastram.
- **Iniciar conversa** nunca cria contato nem envia mensagem; exige exatamente
  um entre `chatId` (`@c.us`) e `telefone`; conflito → `409
  responsavel_diferente`. Origens: painel **Contatos** do Kanban, bloco "Nova
  conversa com número" e botão **Conversar** dos **integrantes do grupo**
  (envia pelo `numeroId` do atendimento de grupo; `@lid` sem telefone
  utilizável não habilita).
- **Finalizar** é terminal e silencioso; o Resolver envia o encerramento
  quando `enviarEncerramento=true`; a automação leva `Resolvido` → `Arquivado`
  após a janela (padrão 15 min, `0` desliga), exceto grupos.
- Telefone: chave canônica sem o nono dígito; cartão/envio com DDI 55 e o 9. Em
  `@lid`/grupo só o `TelefoneResolvido` vale e é display, não identidade.
- `Mensagem.Tipo` é string livre (texto, `image`, `audio`, `video`, `gif`,
  `contact`). Detalhe em [02](../referencia/whatsapp/02-regras-e-contratos-do-modulo.md).

## 8. Autorização

- Atendimento, grupos, IA e auditoria: `perm:aba.suporte.whatsapp:Ler`;
  métricas, respostas rápidas e números: `.metricas`, `.respostas-rapidas`,
  `.numeros`.
- Admin/config/SLA/retenção e escrita de etiquetas/IA: `Administrador`.
  Frontend usa `permissaoGuard`/`adminGuard`; webhook é anônimo com HMAC e
  dedupe por `X-OpenWA-Idempotency-Key`.

## 9. Onde alterar

Fluxo da tela em
[10.7](../referencia/frontend/11-10-fluxos-de-dados/07-10-7-atendimento-whatsapp.md);
tabela completa em
[01](../referencia/whatsapp/01-servicos-e-mapeamento.md).

| Intenção | Backend | Frontend | Teste |
|---|---|---|---|
| “transição de status” | `AtendimentoService.CicloVida.AssumirAsync` | `pages/atendimento/atendimento-whatsapp.component.ts` | `WhatsAppAtendimentoTests.cs` |
| “vincular contato / iniciar conversa” | `ContatoVinculoService.VincularAsync` (ramo `CriarEVincularAsync`), `AtendimentoService.IniciarConversaAsync` | `cliente-context-panel` + `vincular-empresa-modal` (conversa), `pages/admin-contatos` (aba "Sem contato"), `iniciar-conversa-modal` | `WhatsAppContatoVinculoTests.cs`, `WhatsAppIniciarConversaTests.cs` |
| “conversar com integrante do grupo” | `AtendimentoService.IniciarConversaAsync` (mesmo endpoint) | `grupo-integrantes-modal` (recebe `[numeroId]` do `cliente-context-panel`); `WhatsAppService.iniciarConversa` | `WhatsAppIniciarConversaTests.cs`, `grupo-integrantes-modal.component.spec.ts` |

## 10. Contratos que não podem quebrar

- Status é `byte`; a ordem quebra a fila. Webhook público precisa validar
  assinatura; ação sensível grava auditoria.
- Mídia enviada: lista fechada de MIME (imagem sem SVG, áudio, `video/mp4`,
  `video/webm`, `image/gif`), ≤ 10 MB (request 12 MB); documento arbitrário
  recusado.
- Mídia persistida só sai por `GET /mensagens/{id}/midia` (blob autenticado);
  foto por proxy autenticado. Nenhuma URL pública.
- O 409 `responsavel_diferente` vale para texto, mídia, contato e iniciar
  conversa, com `responsavelNome` e `atendimentoId`.
- Contratos completos em
  [17](../referencia/endpoints/whatsapp/17-observacoes-de-contrato.md),
  [18](../referencia/endpoints/whatsapp/18-observacoes-de-contrato-gateway-e-rodadas.md)
  e [02](../referencia/whatsapp/02-regras-e-contratos-do-modulo.md).

## 11. Armadilhas conhecidas

- Timestamp do gateway: use `WhatsAppTimestamp`; a fila do webhook garante
  idempotência.
- Cartão montado com a chave canônica (sem o 9) = número que não disca.

## 12. Testes

Suíte backend `WhatsApp*` e specs do frontend; contagens
e cenários em
[03](../referencia/whatsapp/03-testes-pendencias-e-operacao.md). QA:
`docs/referencia/qa/17-12-atendimento-whatsapp/INDEX.md` (WA-01–128).

## 13. Orçamento de leitura

Só a seção 9 e o teste (1 ficha, 4 trechos, ≤ 12 tool calls).

## 14. Pendências

- **Contatos e iniciar conversa:** falta exercitar no navegador o **Vincular
  empresa pela conversa** (nome obrigatório, 200, 409 identificado, 400 chat
  inexistente) e o iniciar conversa (200, 409, 400, 404) com sessão real.
- **Reset de 2026-10-30:** gateway sem sessão; falta reconectar um número e
  revalidar; `@lid` resolvido sob demanda após a reconexão.
- **Reações (Fase 8):** evidência do gateway informal; `messages/react`,
  `GET .../reactions` e `message.reaction` seguem **NÃO TESTADOS**.
- Histórico de testes e deploys em
  [03](../referencia/whatsapp/03-testes-pendencias-e-operacao.md).
