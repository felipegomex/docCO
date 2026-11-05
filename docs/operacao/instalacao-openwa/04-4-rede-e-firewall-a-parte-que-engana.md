---
id: instalacao-openwa.04
titulo: "4. Rede e firewall — a parte que engana"
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
tokens_estimados: 798
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 4. Rede e firewall — a parte que engana

### 4.1 O `ufw` **não** restringe porta publicada pelo Docker

O `ufw` da VM está ativo com `default deny incoming`, mas as regras dele **não restringem as portas publicadas por containers**. O motivo é a ordem de processamento no kernel:

1. o Docker faz **DNAT** na cadeia `nat/PREROUTING` (a porta da VM vira a porta do container);
2. esse tráfego **não** passa pela cadeia `INPUT` do `ufw` — passa pela `FORWARD`;
3. sem uma barreira explícita, a `2785` fica acessível a **toda a rede**.

Por isso, as regras do `ufw` podem estar corretas e ainda assim não dar segurança nenhuma nesse caso: o sintoma é a porta continuar aberta mesmo com `deny incoming`.

### 4.2 A correção: cadeia `DOCKER-USER`

O ponto de corte correto é a cadeia **`DOCKER-USER`** (filter/`FORWARD`), que o Docker percorre antes de encaminhar o tráfego ao container:

```bash
iptables -I DOCKER-USER -p tcp --dport 2785 -s 192.168.2.130 -j ACCEPT
iptables -I DOCKER-USER -p tcp --dport 2785 -s 192.168.2.254 -j ACCEPT
iptables -A DOCKER-USER -p tcp --dport 2785 -j DROP
```

As duas regras de `ACCEPT` vêm **antes** do `DROP` (por isso o `-I`): o `DROP` é o fim da cadeia para todo o resto.

Verificação (2026-10-21):

| Origem | Resultado |
|---|---|
| `192.168.2.254` | `200` em `/api/health` — liberada |
| `192.168.2.130` | liberada (mesma regra) |
| `192.168.2.5` (estação não autorizada) | **bloqueada** |

### 4.3 Persistência das regras

O Docker **recria** a cadeia `DOCKER-USER` no boot, o que apagaria regras aplicadas à mão. Por isso as regras foram persistidas em `/etc/iptables/rules-openwa.sh` e aplicadas por uma unidade systemd habilitada:

```ini
# /etc/systemd/system/openwa-firewall.service
[Unit]
Description=Regras de firewall do gateway OpenWA (cadeia DOCKER-USER)
After=docker.service
Requires=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/etc/iptables/rules-openwa.sh

[Install]
WantedBy=multi-user.target
```

```bash
chmod 700 /etc/iptables/rules-openwa.sh
systemctl daemon-reload
systemctl enable --now openwa-firewall.service
```

### 4.4 Pendência conhecida — as outras portas publicadas na VM

A mesma falha do `ufw` vale para **todas** as portas publicadas por container naquela VM. Além da `2785`, estão publicadas a **3000**, a **2222** e a **5678** (serviços já existentes do host), e **nenhuma** delas tem barreira equivalente na `DOCKER-USER`. **A decisão de apertá-las ou não é do dono do projeto** — até lá, elas seguem expostas como estavam.

### 4.5 O lado do servidor IIS

No **servidor IIS** do ambiente (Windows, PowerShell administrativo), a `1009` (backend) é liberada **apenas** para a VM do gateway:

```powershell
New-NetFirewallRule -DisplayName "OpenWA -> backend 1009 (webhook)" -Direction Inbound -Protocol TCP -LocalPort 1009 -RemoteAddress 192.168.2.253 -Action Allow
```

