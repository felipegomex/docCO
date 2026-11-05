---
id: 20-19-gateway-openwa-canal-de-whatsapp.indice
titulo: "19. Gateway OpenWA (canal de WhatsApp)"
tipo: operacao
area: diagnostico
sistema: central
tags: [whatsapp, openwa, gateway, erro]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 300
ficha: docs/modulos/whatsapp.md
---

# 19. Gateway OpenWA (canal de WhatsApp)

> Índice do documento dividido. Documento original: [20-19-gateway-openwa-canal-de-whatsapp.md](../20-19-gateway-openwa-canal-de-whatsapp.md).
> Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

> Parte de [docs/operacao/troubleshooting.md](../../troubleshooting.md).


O canal de atendimento WhatsApp tem **duas pontas**: o **gateway OpenWA** (serviÃ§o na VM Ubuntu `192.168.2.253`, porta `2785`) e a **Central** (backend ASP.NET Core, porta `1009`, em `192.168.2.130` na homologaÃ§Ã£o e `192.168.2.254` na produÃ§Ã£o). O navegador nunca fala com o gateway: tudo passa pela API da Central. O runbook de instalaÃ§Ã£o estÃ¡ em [`instalacao-openwa.md`](../../instalacao-openwa/INDEX.md).

O gateway **está instalado e em operação** (2026-10-21): OpenWA v0.25.0 em **Docker** na VM `192.168.2.253`, imagem oficial fixada, porta `2785` publicada no IP da VM e restrita por firewall. O **piloto rodou na homologação** (`.130`) com número pareado e mensagens reais, e a **produção (`.254`) foi publicada em 2026-10-26**. Depois do **reset de dados de 2026-10-27** (§19.11) o gateway ficou **sem sessões** (`GET /api/sessions` = `[]`): **reconecte um número antes da validação manual**. A instalação nativa no Windows Server `192.168.2.254` foi **descartada** e o piloto parcial foi removido do host. Runbook completo em [`instalacao-openwa.md`](../../instalacao-openwa/INDEX.md).

Antes de investigar cÃ³digo, descarte as hipÃ³teses de ambiente â€” sÃ£o as que quebraram de fato na instalaÃ§Ã£o.

## Partes

- [19.1 `npm` e GitHub falham: DNS nÃ£o configurado no host](./01-19-1-npm-e-github-falham-dns-na-o-configurado-no-host.md)
- [19.2 Todo envio falha com `the engine returned no message`: *patchers* do `whatsapp-web.js` nÃ£o aplicados](./02-19-2-todo-envio-falha-com-the-engine-returned-no-message-pat.md)
- [19.3 O serviÃ§o nÃ£o sobe ou a API nÃ£o responde](./03-19-3-o-servia-o-na-o-sobe-ou-a-api-na-o-responde.md)
- [19.4 O webhook do gateway Ã© recusado](./04-19-4-o-webhook-do-gateway-a-recusado.md)
- [19.4.1 Aviso de sistema da Meta vira atendimento](./05-19-4-1-aviso-de-sistema-da-meta-vira-atendimento.md)
- [19.5 A fila de webhook cresce e nada Ã© processado](./06-19-5-a-fila-de-webhook-cresce-e-nada-a-processado.md)
- [19.6 Checklist do canal](./07-19-6-checklist-do-canal.md)
- [19.7 Porta publicada pelo Docker ignora o `ufw`](./08-19-7-porta-publicada-pelo-docker-ignora-o-ufw.md)
- [19.8 Chave da API do OpenWA com valor vazado no log](./09-19-8-chave-da-api-do-openwa-com-valor-vazado-no-log.md)
- [19.11 Reset dos dados do módulo WhatsApp](./12-19-11-reset-dos-dados-do-modulo-whatsapp.md)
- [19.12 Número não exclui / conflito de FK (grupos e participantes)](./13-19-12-numero-nao-exclui-conflito-de-fk-grupos-e-participante.md)
- [19.13 Protocolo/foto/telefone do contato — coluna ausente após publicar as Fases 0/1](./14-19-13-protocolo-foto-telefone-do-contato-coluna-ausente-apos.md)
- [19.14 Atendimento Resolvido não finaliza sozinho (Fase 2)](./15-19-14-atendimento-resolvido-nao-finaliza-sozinho-fase-2.md)
- [19.15 Foto do contato não aparece (Fase 3)](./16-19-15-foto-do-contato-nao-aparece-fase-3.md)
