---
id: instalacao-openwa.08
titulo: "8. Variáveis do backend (Central)"
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
tokens_estimados: 604
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 8. Variáveis do backend (Central)

Todas por **variável de ambiente** no servidor IIS do ambiente — nada disso entra no `appsettings.json` versionado:

| Variável | Valor | Efeito |
|---|---|---|
| `WhatsApp__Habilitado` | `true` | liga o módulo; com `false` o receptor responde `503` e o worker não processa nada |
| `WhatsApp__BaseUrl` | `http://192.168.2.253:2785/api` | endereço do gateway (o `appsettings.json` versionado traz `http://127.0.0.1:2785/api`, herdado do desenho antigo — precisa ser sobrescrito) |
| `WhatsApp__ApiKey` | chave do gateway | header `X-API-Key` em toda chamada; **nunca** versionada |
| `WhatsApp__WebhookSecret` | segredo do webhook | valida o HMAC-SHA256 do corpo cru; **é o mesmo valor** registrado no gateway |
| `WhatsApp__WebhookUrl` | `http://192.168.2.130:...` (HML) ou `http://192.168.2.254:...` (PROD) | URL que o backend registra no gateway; **sem esta variável o gateway aponta para o loopback e nenhum evento chega** |

Confira o que está efetivamente no servidor, sem imprimir valor: a impressão digital (SHA-256, 16 caracteres) do valor lido no `.env` da VM e do valor da variável de ambiente **no servidor** precisam coincidir.

### 8.1 A chave da API usada pela Central

O gateway tem chaves de **escopo restrito** (por sessão, por chat) e chaves de **escopo amplo**. A Central usa uma chave **`operator`** de escopo amplo, porque **cria a sessão dinamicamente** ao cadastrar o número — uma chave por sessão exigiria manutenção a cada cadastro.

Procedimento:

```bash
curl -s -X POST http://127.0.0.1:2785/api/auth/api-keys \
  -H "X-API-Key: SUA_CHAVE_ADMIN" \
  -H "Content-Type: application/json" \
  -d '{"name":"central-operacao","role":"operator"}'
```

A chave ADMIN inicial **só é exibida uma vez**, no log do primeiro boot. A chave de uso da Central foi criada por `POST /api/auth/api-keys` e **trocada logo depois**, porque o valor havia aparecido no log do terminal. Se um valor de chave aparecer em log, terminal ou histórico: **apague a chave e crie outra** (diagnóstico em [`troubleshooting.md`](../troubleshooting.md) §19.8).

