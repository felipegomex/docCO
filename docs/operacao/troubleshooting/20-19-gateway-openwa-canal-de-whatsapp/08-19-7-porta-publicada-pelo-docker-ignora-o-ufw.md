---
id: 20-19-gateway-openwa-canal-de-whatsapp.08
titulo: "19.7 Porta publicada pelo Docker ignora o `ufw`"
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
tokens_estimados: 533
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.7 Porta publicada pelo Docker ignora o `ufw`

**Sintoma.** A `2785` responde a partir de **qualquer** host da rede, mesmo com o `ufw` ativo e `default deny incoming`. As regras do `ufw` existem, estÃ£o corretas e **nÃ£o bloqueiam nada**.

**Causa.** O `ufw` filtra a cadeia `INPUT`, mas o trÃ¡fego de container **nÃ£o passa por ela**:

1. o Docker faz **DNAT** na cadeia `nat/PREROUTING` (porta da VM â†’ porta do container);
2. o pacote segue pelo caminho de **encaminhamento**, que passa pela cadeia `FORWARD` â€” e a `INPUT` do `ufw` nÃ£o Ã© consultada;
3. sem uma barreira explÃ­cita, a porta publicada Ã© alcanÃ§Ã¡vel por toda a rede.

**SoluÃ§Ã£o â€” cadeia `DOCKER-USER`** (filter/`FORWARD`), que o Docker percorre antes de encaminhar ao container:

```bash
iptables -I DOCKER-USER -p tcp --dport 2785 -s 192.168.2.130 -j ACCEPT
iptables -I DOCKER-USER -p tcp --dport 2785 -s 192.168.2.254 -j ACCEPT
iptables -A DOCKER-USER -p tcp --dport 2785 -j DROP
```

Os dois `ACCEPT` vÃªm **antes** do `DROP` (por isso `-I`, nÃ£o `-A`).

âš ï¸ **PersistÃªncia.** O Docker **recria** a cadeia `DOCKER-USER` no boot, o que apagaria regras aplicadas Ã  mÃ£o. Grave as regras em `/etc/iptables/rules-openwa.sh` e aplique por uma unidade systemd habilitada (`openwa-firewall.service`, `After=docker.service`, `ExecStart=/etc/iptables/rules-openwa.sh`), com `systemctl enable --now`.

**VerificaÃ§Ã£o (2026-10-21):** `192.168.2.254` â†’ `200` em `/api/health`; `192.168.2.5` (estaÃ§Ã£o nÃ£o autorizada) â†’ **bloqueada**.

âš ï¸ **As outras portas publicadas na VM tÃªm o mesmo problema:** `3000`, `2222` e `5678` (serviÃ§os jÃ¡ existentes do host) **nÃ£o** tÃªm barreira equivalente na `DOCKER-USER`. **A decisÃ£o de apertÃ¡-las ou nÃ£o Ã© do dono do projeto** â€” atÃ© lÃ¡, elas seguem expostas como estavam.

