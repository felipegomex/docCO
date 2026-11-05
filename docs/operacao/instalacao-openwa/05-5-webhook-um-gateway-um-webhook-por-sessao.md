---
id: instalacao-openwa.05
titulo: "5. Webhook: um gateway, um webhook por sessão"
tipo: operacao
area: operacao
sistema: central
tags: [openwa, gateway, whatsapp, docker]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 634
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 5. Webhook: um gateway, um webhook por sessão

Na prática da Central, há **um webhook ativo por sessão**: quem aponta a URL é o **backend**, no momento em que o número é cadastrado (`POST /api/v1/whatsapp/numeros` → `RegistrarWebhookAsync` → `POST /api/sessions/{id}/webhooks`), com o segredo e `retryCount = 3`.

A URL vem de `WhatsApp:WebhookUrl` — cujo **padrão no código é `http://127.0.0.1:1009/api/v1/whatsapp/webhook/openwa`**, herança do desenho antigo (gateway e backend no mesmo host). **Esse padrão está errado para a topologia atual**: o gateway está em outra máquina, então a variável **precisa** ser sobrescrita (§7).

| Ambiente | URL do webhook | Servidor da Central |
|---|---|---|
| Homologação | `http://192.168.2.130:1009/api/v1/whatsapp/webhook/openwa` | `192.168.2.130` |
| Produção | `http://192.168.2.254:1009/api/v1/whatsapp/webhook/openwa` | `192.168.2.254` |

**A virada de ambiente é só configuração**: troca-se `WhatsApp__WebhookUrl` no servidor e o backend passa a registrar o webhook novo. **Não reinstale o gateway.**

⚠️ **Risco de deixar os dois apontados ao mesmo tempo.** O OpenWA aceita **vários** webhooks por sessão (`WEBHOOK_MAX_PER_SESSION=16`), e a lista de endereços liberados no `SSRF_ALLOWED_HOSTS` cobre os dois IPs. Se homologação e produção ficarem **simultaneamente** apontados para a mesma sessão:

1. **cada evento é entregue aos dois backends** — o mesmo atendimento nasce em duas filas, em duas bases, com dois históricos que ninguém reconcilia;
2. a homologação passa a criar e resolver atendimentos de **produção** (e o inverso), poluindo métricas e disparando mensagem real para cliente real;
3. os dois backends tentam **gerenciar a mesma sessão** (criar/parar/reconectar e, no registro do webhook, `POST` duplicado) — um `POST /api/sessions` a partir dos dois lados cria **duas sessões** para o mesmo número, e o WhatsApp encerra o dispositivo mais antigo.

**Regra:** no máximo **um** backend ativo por sessão. Na virada, desative o webhook do ambiente antigo (remova ou desative o registro correspondente na sessão do gateway) **antes** de ativar o novo, e confira `GET /api/sessions/{id}/webhooks`: deve haver **uma** URL ativa, a do ambiente em uso.

