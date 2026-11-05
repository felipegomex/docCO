---
id: 20-19-gateway-openwa-canal-de-whatsapp.01
titulo: "19.1 `npm` e GitHub falham: DNS nÃ£o configurado no host"
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
tokens_estimados: 313
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.1 `npm` e GitHub falham: DNS nÃ£o configurado no host

| Sintoma | Causa | SoluÃ§Ã£o |
|---|---|---|
| `npm ci` falha com `ETIMEDOUT`/`ENOTFOUND` ao baixar o registry; `git clone`/`curl` do GitHub nÃ£o resolve | **O adaptador de rede do host estava sem servidor DNS configurado.** A saÃ­da por IP funcionava, mas a **resoluÃ§Ã£o de nomes** nÃ£o â€” e Ã© por nome que o npm e o GitHub sÃ£o alcanÃ§ados. O sintoma Ã© enganoso: parece "sem internet", mas hÃ¡ internet, sÃ³ nÃ£o hÃ¡ DNS | No servidor Windows, confira e configure o servidor DNS do adaptador: `192.168.2.1, 8.8.8.8` (roteador local e DNS pÃºblico, nesta ordem) â€” verificaÃ§Ã£o com `Get-DnsClientServerAddress -AddressFamily IPv4` e `Resolve-DnsName github.com`. Na VM Ubuntu, o equivalente Ã© `resolvectl status` e `getent hosts github.com`; com `systemd-resolved`, defina em `/etc/systemd/resolved.conf` (`DNS=192.168.2.1 8.8.8.8`) e reinicie o serviÃ§o |

