---
id: instalacao-openwa.09
titulo: "9. Prova técnica — o que foi verificado, o que falta e o ciclo de vida da sessão"
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
tokens_estimados: 1648
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 9. Prova técnica — o que foi verificado, o que falta e o ciclo de vida da sessão

### 9.1 Verificado em 2026-10-21

| # | Item | Evidência |
|---:|---|---|
| 1 | Gateway no ar em Docker na VM `.253` | container `openwa-api` em execução; `restart: unless-stopped` |
| 2 | API responde | `GET http://192.168.2.253:2785/api/health` → `200` a partir de `192.168.2.254` |
| 3 | Porta restrita | `192.168.2.5` (estação não autorizada) **bloqueada** pela cadeia `DOCKER-USER` |
| 4 | Schema aplicado | 7 tabelas `tbwhatsapp_*`, 11 FKs, 7 `CHECK`, 89 colunas em `dbBUSINESS_HML`; reaplicação = no-op |
| 5 | Publicação do backend na homologação | `validate.ps1` verde → package com `deploy.ps1 -Ambiente HML` → publicação com `-Publicar $true -Backup $true -UsuarioRemoto 'JCASRV-SUP' -SenhaRemota <segredo>` |
| 6 | Smoke da homologação | `smoke.ps1 -Servidor 192.168.2.130` verde: frontend `200`, swagger `200`, runtime-config `200` |
| 7 | Webhook ponta a ponta (HML) | assinatura inválida → `401`; sem assinatura → `401`; assinatura válida → `200 {"aceito":true,"duplicado":false}`; reentrega da **mesma** `idempotencyKey` → `200 {"aceito":true,"duplicado":true}` |
| 8 | Fila durável | a linha foi gravada em `tbwhatsapp_webhook_fila` com `estado` = processado e sem erro |
| 9 | Central publicada na homologação | backend `:1009` e frontend `:1010` no `.130`, `smoke.ps1` verde (itens 5 e 6) |
| 10 | Número pareado (piloto, 2026-10-21) | sessão `central-wa-7`, status `ready`, telefone `558592644863` ("Felipe Gomes Corporativo") gravado em `tbwhatsapp_numero` — sessão removida no reset de 2026-10-27 |
| 11 | Webhook registrado (um só) | `GET /api/sessions/{id}/webhooks` → **uma** URL ativa, a do ambiente em uso (§5) |
| 12 | Mensagem **recebida** | mensagem real do WhatsApp chegou pelo webhook, foi registrada e **criou atendimento na fila** |
| 13 | Primeira resposta | a primeira resposta assumiu o atendimento; houve **resolução** e **reabertura** |
| 14 | Tela de números | reflete o status **sozinho** (a cada 15 s) e fecha o painel do QR quando a sessão sai de `qr_ready` |

### 9.2 Pendente

| # | Item | Como verificar |
|---:|---|---|
| 15 | **Reconectar um número** | gateway **sem sessões** após o reset de 2026-10-27: cadastrar o número em `/whatsapp/numeros` (cria + inicia + registra webhook), ler o QR e confirmar `ready` antes do QA manual. Em produção (`.254`), mantenha o webhook do `.130` desativado (§5): no máximo **um** backend ativo por sessão |
| 16 | Migrações do gateway | o gateway **não** aplica migrations no boot: quem roda é o operador, na atualização de versão (§10.2). O container, por si, **sobe sozinho** com o daemon do Docker (`restart: unless-stopped`) |

### 9.3 Ciclo de vida da sessão no gateway

O que a Central faz no gateway quando o administrador cadastra e quando remove um número — verificado no gateway da VM em 2026-10-21.

| Passo | O que acontece no gateway | Consequência para o administrador |
|---|---|---|
| **Cadastrar o número** | o backend **cria** a sessão (`POST /api/sessions`, nome `central-wa-<id>`) e **a inicia** em seguida (`POST /api/sessions/{id}/start`); depois registra o webhook | **o QR fica disponível logo em seguida** — não é preciso iniciar a sessão à mão. Se a criação da sessão falhar, o número fica cadastrado na Central com status `failed` e o administrador reconecta depois |
| **Ler o QR** | a sessão passa a `qr_ready`; depois de lido no celular, o gateway avisa por webhook (`session.authenticated`) e a sessão vai a `ready` | o **telefone** é gravado pela Central a partir desse evento (`tbwhatsapp_numero.WAT_Numero`) — o cadastro é atualizado **por webhook**, fora da Central, e é por isso que a tela de números precisa se atualizar sozinha |
| **Iniciar** (botão da tela) | `POST /api/sessions/{id}/start` | usado para reconectar uma sessão que ficou `disconnected` |
| **Parar** (botão da tela) | `POST /api/sessions/{id}/stop` | a sessão para de consumir recursos, mas **continua cadastrada** no gateway |
| **Remover o número** | o backend **remove a sessão**: `DELETE /api/sessions/{id}` → **`204`** | a sessão **some do gateway** — não fica resíduo. Se o gateway não tiver a rota (ou estiver fora do ar), a Central **cai para o `stop`** e a desativação do cadastro acontece de qualquer forma |

⚠️ **O que a remoção NÃO faz:** nenhum atendimento, mensagem ou evento é apagado. O histórico do canal é do **SQL Server da Central** (`tbwhatsapp_*`) e **permanece** — o que sai do gateway é apenas a sessão.

⚠️ **`BaseUrl` sem barra final manda o pedido para o endereço errado.** As rotas do cliente são relativas (`sessions`) e o .NET resolve caminho relativo contra o `BaseAddress`: `WhatsApp__BaseUrl=http://192.168.2.253:2785/api` **sem barra final** faria o pedido sair para `/sessions` em vez de `/api/sessions`, e o gateway responderia `404` em uma rota que existe. O backend **normaliza a barra** (`OpenWaClient.NormalizarBaseUrl`), então o valor configurado não precisa vir com barra — mas vale conferir a URL real no log do gateway. Diagnóstico em [`troubleshooting.md`](../troubleshooting.md) §19.9.1.

### 9.4 Reenfileirar a fila de webhook

A fila de webhook do gateway é processada pela Central, no SQL Server (`tbwhatsapp_webhook_fila`). Quando o backend morre no meio do processamento (redeploy, recycle do pool do IIS), a linha fica **presa em "processando"** e o evento se perde. O worker agora **se auto-recupera** (§19.10.2 do [`troubleshooting.md`](../troubleshooting.md)); para forçar na mão:

```sql
-- Reenfileirar as linhas presas em "processando" (WAT_Estado = 1).
UPDATE tbwhatsapp_webhook_fila
SET WAT_Estado = 0,                  -- 0 = pendente: volta para a fila
    WAT_Tentativas = 0,              -- zera o contador (o worker desiste em 6 tentativas)
    WAT_ProximoProcessamentoEm = NULL,
    WAT_Erro = NULL
WHERE WAT_Estado = 1;
```

Estado da fila antes de mexer:

```sql
SELECT WAT_Id, WAT_Evento, WAT_Estado, WAT_Tentativas, WAT_RecebidoEm, WAT_Erro
FROM tbwhatsapp_webhook_fila
WHERE WAT_Estado IN (1, 3)
ORDER BY WAT_Id;
```

⚠️ O reprocessamento é **idempotente**: o índice único `(WAT_ChatId, WAT_ExternalId)` de `tbwhatsapp_mensagem` impede mensagem duplicada, e a chave única da fila impede o reprocessamento do mesmo webhook.

