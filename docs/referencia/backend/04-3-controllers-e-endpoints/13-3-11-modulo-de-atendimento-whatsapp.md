---
id: 04-3-controllers-e-endpoints.13
titulo: "3.11 Módulo de atendimento WhatsApp"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1448
ficha: docs/modulos/whatsapp.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 3.11 Módulo de atendimento WhatsApp

Quinze controllers na pasta `Controllers/WhatsApp`, **76 operações** `[Http*]`, todas versionadas em `api/v{version:apiVersion}/whatsapp`. O catálogo detalhado (corpos, autorização e respostas) está em [`endpoints/whatsapp.md`](../../endpoints/whatsapp/INDEX.md). A **Fase 09** não criou controller nem rota REST: acrescentou o hub SignalR `/hubs/whatsapp` (`Hubs/WhatsAppHub.cs`), que fica **fora** da contagem de `[Http*]` e não substitui nenhum endpoint — os métodos do hub são notificação de saída (§4.5.3). Categorias e Tags (Fase D) foram removidos em 2026-10-23; etiquetas (Fase 06) são um conceito novo, com tabelas próprias. A Fase 08 acrescentou o controller administrativo de contatos (duas leituras em `/admin/contatos`, só administrador), sem escrita nova. A **F6** acrescentou `POST /atendimentos/{id}/visto` (status **6**), a única rota nova desde a Fase 08 até a Fase 3. As **Fases 0/1 do plano de evolução (2026-10-27)** não criaram rota: o `GET /atendimentos` ganhou o filtro `protocolo`, o resumo ganhou `protocolo`/`contatoTelefone`, e o schema ganhou protocolo computado, foto/telefone resolvido do contato e dois parâmetros (§3.11 e §4.5.1.2). A **Fase 3** acrescentou `GET /contatos/{id}/foto` (proxy autenticado da foto), a única rota nova posterior à F6. A **Fase 4** também não criou rota: acrescentou `etiquetaId` ao `GET /metricas` e o bloco `porEtiqueta` na resposta.

| Controller | Rota base | Operações | Autorização |
|---|---|---:|---|
| `WhatsAppAtendimentosController` | `/api/v1/whatsapp/atendimentos` | 26 | `[Authorize]`; **404** tanto para "não existe" quanto para "existe, mas não é seu" — responder 403 confirmaria a existência do card de outro; arquivamento manual **só administrador**; `visto` por qualquer atendente |
| `WhatsAppNumerosController` | `/api/v1/whatsapp/numeros` | 10 | `[Authorize]`; leitura por qualquer atendente, escrita **só administrador** (`IsInRole("Administrador")` ou claim `eh_admin == "true"` ou claim `perfil_id == "A"`) |
| `WhatsAppWebhookController` | `/api/v1/whatsapp/webhook` | 1 | **`[AllowAnonymous]`** — quem chama é o gateway; a autenticação é a assinatura HMAC do corpo cru |
| `WhatsAppMetricasController` | `/api/v1/whatsapp` | 3 | `[Authorize]`; **qualidade só administrador** |
| `WhatsAppRespostasRapidasController` | `/api/v1/whatsapp/respostas-rapidas` | 5 | `[Authorize]`; leitura por atendente, escrita **só administrador** |
| `WhatsAppConfiguracaoController` | `/api/v1/whatsapp` | 4 | `[Authorize(Roles = "Administrador")]`; checklist, janela da finalização automática (`GET`/`PUT`) e monitor |
| `WhatsAppSlaController` | `/api/v1/whatsapp/sla` | 4 | `[Authorize]`; leitura por atendente, gravação e histórico **só administrador** |
| `WhatsAppRetencaoController` | `/api/v1/whatsapp/retencao` | 3 | `[Authorize]`; **tudo só administrador** |
| `WhatsAppContatosController` | `/api/v1/whatsapp/contatos` | 7 | `[Authorize]`; ficha/typeahead/lista/diagnóstico por atendente; **PATCH do vínculo** por qualquer atendente autorizado no canal para contato **pendente** — trocar/remover vínculo de contato **identificado** exige admin **revalidada no service**; unificar **só administrador** |
| `WhatsAppGruposController` | `/api/v1/whatsapp/grupos` | 3 | `perm:aba.suporte.whatsapp:Ler`; **só leitura** — grupos e participantes nascem do webhook |
| `WhatsAppMensagensController` | `/api/v1/whatsapp/mensagens` | 1 | `[Authorize]`; download de mídia sob demanda |

**Novos controllers (Fases F, G, Fase 06 e Fase 08 do novo ciclo):**

| Controller | Rota base | Operações | Autorização |
|---|---|---:|---|
| `WhatsAppAuditoriaController` | `/api/v1/whatsapp/auditoria` | 1 (GET) | Leitura: `perm:aba.suporte.whatsapp:Ler` + **só administrador** (controller verifica `EhAdministrador`) |
| `WhatsAppIaController` | `/api/v1/whatsapp/ia` | 3 (GET/PUT config, GET assistente) | Config: **só administrador**; Assistente: `perm:aba.suporte.whatsapp:Ler` |
| `WhatsAppEtiquetasController` | `/api/v1/whatsapp/etiquetas` | 3 (GET/POST/PATCH) | `[Authorize]`; **GET por qualquer atendente**; **POST/PATCH do catálogo só administrador** (`EhAdministrador` no controller — esconder o botão não é controle de acesso) |
| `WhatsAppAdminController` | `/api/v1/whatsapp/admin` | 2 (GET contatos, GET histórico) | `[Authorize]`; **todas as rotas exigem administrador** (`EhAdministrador` no controller → `403`). A escrita de contato **não** está aqui: continua no `PATCH /contatos/{id}` da Fase 01 |

A expressão de administrador é a mesma em todos os controllers: `IsInRole("Administrador")` **ou** claim `eh_admin == "true"` **ou** claim `perfil_id == "A"`.

Três decisões estruturais do módulo:

1. **O corpo nunca define o autor.** O `OperadorId` do corpo existe apenas como **destino** de transferência; quem age vem do claim `NameIdentifier` (fallback `sub`).
2. **Falha do gateway não é erro do usuário:** `OpenWaException` vira **502** (`Bad Gateway`), não 400 — 400 diria "o pedido está errado", e o pedido está certo.
3. **404 em vez de 403** nos atendimentos: a resposta é a mesma para "não existe" e "existe, mas não é seu", para não revelar a existência do card de outro.

O receptor de webhook lê o corpo **cru** (buffer) antes de qualquer desserialização, impõe o limite de **1 MiB** (`RequestSizeLimit` + leitura manual, com `413` acima do limite) e entrega o envelope à **fila durável**; a resposta é rápida de propósito, porque o gateway dá timeout em 10 s e repete a entrega.

