---
id: whatsapp.18
titulo: "Observações de contrato — gateway, foto, grupos e rodadas"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints, gateway, openwa, foto, grupos]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 1400
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).
> Continuação de [Observações de contrato](./17-observacoes-de-contrato.md).

## Observações de contrato — gateway, foto, grupos e rodadas

- O navegador **nunca** fala com o OpenWA: toda a integração passa pelo backend.
- **Foto e telefone do contato no gateway.** O backend consulta `GET /api/sessions/{sessao}/contacts/{chatId}/profile-picture` (`OpenWaFotoPerfilDto.Url`) e `GET /api/sessions/{sessao}/contacts/{chatId}/phone` (`OpenWaTelefoneDto.Number`). As duas rotas são **tolerantes**: `404` vira `null` (sem foto/sem telefone resolvido) e falha de rede, timeout ou resposta inesperada também viram `null`, com log — nunca erro para a tela. A URL devolvida pela primeira é temporária e só é usada **no servidor** — o navegador recebe a imagem pelo `GET /api/v1/whatsapp/contatos/{id}/foto`. No `@lid`, o telefone resolvido é gravado em `WAT_TelefoneResolvido` para exibição, sem trocar a chave de identidade.
- **Rota de grupos do gateway não é endpoint da Central.** Para obter o título real do grupo, o backend chama `GET /api/sessions/{sessaoId}/groups?limit=1000&offset=0` (`IOpenWaClient.ListarGruposAsync`, tolerante a falha: devolve lista vazia) e grava em `tbwhatsapp_grupo` via `IWhatsAppGrupoService.SincronizarAsync`. Para os **integrantes**, chama `GET /api/sessions/{sessaoId}/groups/{chatId}` (`IOpenWaClient.ObterGrupoAsync`), única rota do OpenWA 0.25 que devolve `participants: [{ id, number, name?, isAdmin, isSuperAdmin }]`; a listagem traz apenas a contagem. Qualquer falha vira `null` e vale o cadastro local. O que a Central expõe é apenas `GET /api/v1/whatsapp/grupos` e `GET /api/v1/whatsapp/grupos/{id}/participantes` (leitura, JWT). A sincronização do título ocorre na listagem por número e na chegada de mensagem de grupo, com cache em memória de **5 min por número**; a dos integrantes ocorre na leitura de participantes. Sem schema novo.
- O gateway OpenWA roda **fora** do servidor IIS, na VM Ubuntu `192.168.2.253` — **instalado e em operação** desde 2026-10-21 (runbook em [`operacao/instalacao-openwa.md`](../../../operacao/instalacao-openwa/INDEX.md)): o backend o alcança por `WhatsApp__BaseUrl=http://192.168.2.253:2785/api` e ele entrega o webhook no backend do ambiente por `WhatsApp__WebhookUrl` (`http://192.168.2.130:1009/api/v1/whatsapp/webhook/openwa` na homologação, `http://192.168.2.254:1009/api/v1/whatsapp/webhook/openwa` na produção). Os padrões versionados (`http://127.0.0.1:…`) são herança do desenho antigo, em que o gateway dividia o servidor com o backend.
- **Rodada 2026-11-03 — Contatos e iniciar conversa (sem schema novo).** Novo `POST /atendimentos/iniciar`, `GET /contatos` paginado e `ContatoVinculoService.VincularAsync` unificando `PATCH` e criação; o iniciar **nunca cria contato nem envia mensagem**. Contrato completo em [Atendimentos](./01-atendimentos-atendimentos.md) e [Contatos](./11-contatos-contatos.md).
- **Rodada 2026-11-03 — contato só no vínculo.** O cadastro automático continua removido: o contato é criado só por `POST /contatos/vincular` (`empresaId` e `nome` obrigatórios), agora pela conversa (painel de contexto) ou pela aba "Sem contato" do Admin (`GET /contatos/sem-contato`). O POST exige `perm:aba.suporte.whatsapp:Ler`; reaproveita contato pendente sem renomear; responde `409 CONFLITO_CONTATO_IDENTIFICADO` para chat já identificado e `400` para `chatId` sem conversa. `GET /contatos/foto?chatId=` serve a foto de chat ainda sem contato (`404` → iniciais). `ParticipanteWhatsApp.TelefoneExibicao` (aditivo) prioriza nome do gateway → contato do mesmo `ChatId` → telefone, persistindo o nome descoberto.
- **Rodada 2026-10-30 — telefone, menção, prefixo, busca e envio (sem schema/migration nova).** `TelefoneNormalizador` ganhou `FormatarExibicao` (13 dígitos, com o 9) e `FormatarContato(chatId, resolvido, numero, normalizado)`: em `@lid`/grupo só o resolvido vale; em privado, `resolvido ?? numero ?? normalizado`. `RegistrarEntradaAsync` grava `WAT_Numero` com o telefone de cartão do contato privado; `@lid`/`@g.us` nunca viram número; `WAT_TelefoneResolvido` é display. O novo `ContatoTelefoneResolver` (`IContatoTelefoneResolver`) resolve o `@lid` **sob demanda** em `GET sessions/{sessao}/contacts/{chatId}/phone`, grava o campo e é chamado por `ContatoVinculoService.ObterAsync`; sem sessão ou com falha do gateway a ficha não quebra. `ContatoResumoDto` ganhou `EmpresaId`/`EmpresaNome` (aditivo). `EntradaWhatsApp.MentionedIds` é propagado pelo webhook e o service troca `@<jid>` pelo nome salvo (contato por `ChatId`/`TelefoneNormalizado`, depois participante por `ChatId`); o payload bruto segue na fila. `EnviarMensagemAsync` aplica `*NOME:* ` (primeiro nome do operador em `tboperador`, maiúsculas) antes de persistir e enviar, sem duplicar; nota interna e mídia **não** recebem prefixo. `SolicitarPrivadoAsync` monta o `ChatId` com `NormalizarParaCartao`. O `whatsapp-realtime.service.ts` só aceita `whatsappHubUrl` de **mesma origem** da `apiBaseUrl` (e não localhost em host remoto); senão deriva dela, e `conectar()` aguarda o auth. Backend **470** testes WhatsApp e frontend **266** specs verdes.
