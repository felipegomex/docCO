---
id: whatsapp.indice
titulo: "Endpoints — Controllers de atendimento WhatsApp"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-11-05"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 300
ficha: docs/modulos/whatsapp.md
---

# Endpoints — Controllers de atendimento WhatsApp

> Índice do documento dividido. Documento original: [whatsapp.md](../whatsapp.md).
> Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

Pasta `backend/Central_BackEnd/Controllers/WhatsApp`: **15 classes** e **79 operações** `[Http*]`, contadas a partir dos atributos dos controllers atuais (recontagem de 2026-11-05, com a Fase 01 do novo ciclo — ficha, typeahead de empresas e PATCH do vínculo do contato —, a Fase 06 — etiquetas, investigação e observação do atendimento —, a **Fase 08** — duas leituras administrativas de contatos em `/admin/contatos` —, a **F6** — `POST /atendimentos/{id}/visto`, status `Visto` = 6 —, a **Fase 3** — `GET /contatos/{id}/foto` e a **rodada 2026-11-03** — `GET /contatos/sem-contato`, `POST /contatos/vincular`, `GET /contatos/foto?chatId=` e `GET/PUT /configuracao/finalizacao-automatica`; Atendimentos 26 e Contatos 10; a soma anterior "78" da redação 2026-11-03 não batia com a tabela, que somava 77). Categorias e Tags (Fase D) foram removidas em 2026-10-23. **Todas as rotas são versionadas** (`api/v{version:apiVersion}/whatsapp`); a única particularidade é de **autorização**: o receptor de webhook é `[AllowAnonymous]` — quem chama é o gateway, não um usuário. **A Fase 09 não acrescentou operação REST**: criou o hub SignalR `/hubs/whatsapp` (seção própria abaixo), que é **notificação de saída** e não substitui a API. **As Fases 0 e 1 do plano de evolução (2026-10-27) não criaram rota**: acrescentaram `protocolo` ao filtro/resumo do atendimento, o telefone de exibição do contato e o schema de protocolo/foto (`WhatsApp_Fase8_Idempotente.sql`). **A Fase 3** acrescentou a rota autenticada da foto do contato. **A Fase 4 (métricas por etiqueta) não criou rota**: acrescentou `etiquetaId` ao `GET /metricas` e `porEtiqueta` na resposta. **As Fases 6 e 7 do plano (2026-10-30)**: a Fase 6 (vídeo `mp4`/`webm` e GIF) **não criou rota** — ampliou a whitelist de `POST /atendimentos/{id}/mensagens/midia` no mesmo teto de 10 MB; a Fase 7 acrescentou `POST /atendimentos/{id}/mensagens/contato` (cartão de contato), levando a contagem a **73**. **A Fase 8 (reações)** acrescentou `POST /atendimentos/{id}/mensagens/{mensagemId}/reacao` — **habilitada por configuração desde 2026-10-30**, com evidência do gateway apenas informal —, levando a contagem a **74**. A rodada de **2026-11-01** acrescentou `GET`/`PUT /configuracao/finalizacao-automatica` (janela da automação), chegando a **76**. A rodada de **2026-11-03** removeu `POST /contatos` e `POST /atendimentos/nova-conversa` e acrescentou `GET /contatos/sem-contato`, `GET /contatos/foto?chatId=` e `POST /contatos/vincular` (única porta de criação de contato), chegando a **79** na recontagem de 2026-11-05. Índice do catálogo em [`endpoints.md`](../../endpoints.md); seções por pasta: [`raiz.md`](../raiz/INDEX.md), [`implantacao.md`](../implantacao/INDEX.md), [`dashboard.md`](../dashboard.md) e [`database.md`](../database.md).

Fonte: [WhatsAppAtendimentosController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppAtendimentosController.cs), [WhatsAppNumerosController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppNumerosController.cs), [WhatsAppWebhookController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppWebhookController.cs), [WhatsAppMetricasController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppMetricasController.cs), [WhatsAppRespostasRapidasController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppRespostasRapidasController.cs), [WhatsAppConfiguracaoController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppConfiguracaoController.cs), [WhatsAppSlaController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppSlaController.cs), [WhatsAppRetencaoController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppRetencaoController.cs), [WhatsAppContatosController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppContatosController.cs), [WhatsAppGruposController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppGruposController.cs), [WhatsAppMensagensController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppMensagensController.cs), [WhatsAppAuditoriaController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppAuditoriaController.cs), [WhatsAppIaController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppIaController.cs), [WhatsAppEtiquetasController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppEtiquetasController.cs), [WhatsAppAdminController.cs](../../../../backend/Central_BackEnd/Controllers/WhatsApp/WhatsAppAdminController.cs). Contrato do módulo em [`material-de-projeto/atendimento-whatsapp/README.md`](../../../../material-de-projeto/atendimento-whatsapp/README.md).

Soma por controller:

| Controller | Operações | Rota base | Autorização |
|---|---:|---|---|
| `WhatsAppAtendimentosController` | 26 | `/api/v1/whatsapp/atendimentos` | `[Authorize(Policy = "perm:aba.suporte.whatsapp:Ler")]` na classe; arquivamento manual **só administrador**; etiquetas/investigação/observação/visto/envios (texto, mídia, contato) por **qualquer atendente**; reação também (habilitada por configuração desde 2026-10-30) |
| `WhatsAppNumerosController` | 10 | `/api/v1/whatsapp/numeros` | `[Authorize]` na classe; escrita **só administrador** |
| `WhatsAppWebhookController` | 1 | `/api/v1/whatsapp/webhook` | `[AllowAnonymous]` (HMAC) |
| `WhatsAppMetricasController` | 3 | `/api/v1/whatsapp` | `[Authorize]` na classe; **qualidade só administrador** |
| `WhatsAppRespostasRapidasController` | 5 | `/api/v1/whatsapp/respostas-rapidas` | `[Authorize]`; escrita **só administrador** |
| `WhatsAppConfiguracaoController` | 4 | `/api/v1/whatsapp` | `[Authorize(Roles = "Administrador")]`; checklist, monitor e **janela da finalização automática** com a mesma regra de admin (papel/`eh_admin`/`perfil_id`) |
| `WhatsAppSlaController` | 4 | `/api/v1/whatsapp/sla` | `[Authorize]`; escrita e histórico **só administrador** |
| `WhatsAppRetencaoController` | 3 | `/api/v1/whatsapp/retencao` | `[Authorize]`; **tudo só administrador** |
| `WhatsAppContatosController` | 10 | `/api/v1/whatsapp/contatos` | `[Authorize]`; a aba **"Sem contato"** (`GET /contatos/sem-contato`) é **só administrador**; o **vínculo que cria o contato** (`POST /contatos/vincular`) exige `perm:aba.suporte.whatsapp:Ler` (qualquer atendente, com empresa e nome obrigatórios); unificar e trocar/remover vínculo de contato identificado continuam **só administrador**; foto é proxy autenticado (`GET /contatos/{id}/foto` e `GET /contatos/foto?chatId=`) |
| `WhatsAppGruposController` | 3 | `/api/v1/whatsapp/grupos` | `perm:aba.suporte.whatsapp:Ler` — **só leitura** (o webhook cria a linha; o título real vem da rota de grupos do gateway e os integrantes, do detalhe do grupo) |
| `WhatsAppMensagensController` | 1 | `/api/v1/whatsapp/mensagens` | `[Authorize]` |
| **Novos (Fases F, G):** | | | |
| `WhatsAppAuditoriaController` | 1 | `/api/v1/whatsapp/auditoria` | Leitura: `perm:aba.suporte.whatsapp:Ler` + **só administrador** |
| `WhatsAppIaController` | 3 | `/api/v1/whatsapp/ia` | Config: **só administrador**; Assistente: `perm:aba.suporte.whatsapp:Ler` |
| `WhatsAppEtiquetasController` | 3 | `/api/v1/whatsapp/etiquetas` | `[Authorize]`; **GET por qualquer atendente**; **POST/PATCH do catálogo só administrador** |
| `WhatsAppAdminController` | 2 | `/api/v1/whatsapp/admin` | `[Authorize]` na classe; **todas as rotas só administrador** (`403` sem perfil) |
| **Total** | **79** | — | — |

**Administrador** = `User.IsInRole("Administrador")` **ou** claim `eh_admin == "true"` **ou** claim `perfil_id == "A"` — a mesma expressão usada em `IdeiasController`.

**Operador** = claim `NameIdentifier` (fallback `sub`). **O corpo da requisição nunca define o autor**: o `operadorId` do corpo de transferência é o **destino**, nunca quem age.

## Partes

- [Atendimentos (`/atendimentos`)](./01-atendimentos-atendimentos.md)
- [Etiquetas (`/etiquetas`) — Fase 06](./02-etiquetas-etiquetas-fase-06.md)
- [Mídia das mensagens (`/mensagens`)](./03-midia-das-mensagens-mensagens.md)
- [Números (`/numeros`)](./04-numeros-numeros.md)
- [Webhook (`/webhook`)](./05-webhook-webhook.md)
- [Métricas e saúde](./06-metricas-e-saude.md)
- [Respostas rápidas (`/respostas-rapidas`)](./07-respostas-rapidas-respostas-rapidas.md)
- [Configuração e monitor](./08-configuracao-e-monitor.md)
- [SLA (`/sla`)](./09-sla-sla.md)
- [Retenção de mídia (`/retencao`)](./10-retencao-de-midia-retencao.md)
- [Contatos (`/contatos`)](./11-contatos-contatos.md)
- [Grupos (`/grupos`)](./12-grupos-grupos.md)
- [Admin — contatos (`/admin/contatos`) — Fase 08](./13-admin-contatos-admin-contatos-fase-08.md)
- [Auditoria (`/auditoria`) — Fase F](./14-auditoria-auditoria-fase-f.md)
- [IA (`/ia`) — Fase G](./15-ia-ia-fase-g.md)
- [Tempo real (SignalR) — `/hubs/whatsapp` — Fase 09](./16-tempo-real-signalr-hubs-whatsapp-fase-09.md)
- [Observações de contrato](./17-observacoes-de-contrato.md)
- [Observações de contrato — gateway, foto, grupos e rodadas](./18-observacoes-de-contrato-gateway-e-rodadas.md)
