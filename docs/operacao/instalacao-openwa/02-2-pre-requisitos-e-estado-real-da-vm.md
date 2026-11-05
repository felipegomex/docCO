---
id: instalacao-openwa.02
titulo: "2. Pré-requisitos e estado real da VM"
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
tokens_estimados: 297
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 2. Pré-requisitos e estado real da VM

| Pré-requisito | Estado na VM | Observação |
|---|---|---|
| SO | Ubuntu 24.04.5 LTS | sem versão de kernel fixa |
| Docker | **29.9.1**, já instalado | o `docker-compose.yml` oficial constrói a imagem do fonte; aqui a imagem é **fixada** (§3.2) |
| Docker Compose | **v5.6.0**, já instalado | o `!override` (§3.2) exige Compose ≥ 2.24 |
| RAM | 15 GB | o `whatsapp-web.js` roda um Chromium por sessão: ~**300–500 MB por sessão** (§11) |
| CPU | 4 vCPU | — |
| Disco | 118 GB livres | imagem, dados de sessão e mídia |
| DNS | funcional | clone do projeto e download da imagem dependem de resolução de nomes |
| Acesso SSH | acesso administrativo à VM | mantenha o acesso **por chave** (nunca senha) |

Inventário:

```bash
cat /etc/os-release | head -n 3
free -h
df -h
nproc
docker --version && docker compose version
```

