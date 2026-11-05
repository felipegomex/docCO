---
id: instalacao-openwa.11
titulo: "11. Riscos"
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
tokens_estimados: 459
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 11. Riscos

| Risco | Realidade | Mitigação |
|---|---|---|
| **Bloqueio do número pelo WhatsApp** | o OpenWA conecta por clientes **não oficiais** (`whatsapp-web.js`, `baileys`); o risco de restrição ou ban **não é zero** | número **dedicado** e substituível; aquecimento do número antes de qualquer envio em volume; ritmo baixo de envio; jamais o número principal da empresa |
| **Uso de cliente não oficial** | a licença MIT do código OpenWA não elimina o risco perante as regras do WhatsApp | manter a integração **isolada**, para permitir avaliar a API oficial da Meta; não tratar o gateway como canal autenticado (login/OTP) |
| **RAM por sessão** | `whatsapp-web.js` roda um Chromium por sessão: ~**300–500 MB**; a VM tem 15 GB e o teto configurado é **2 sessões** | começar com **uma** sessão; medir o consumo real antes de conectar números adicionais |
| **Outras portas publicadas na VM sem barreira** | `3000`, `2222` e `5678` estão expostas como estavam (§4.4) | decisão do dono do projeto: apertá-las ou não |
| **`Jwt:Key` em texto claro no `appsettings.json` publicado** | ⚠️ o backend publicado no `.254` tem `Jwt:Key` em texto claro, contrariando a regra do `AGENTS.md` (que exige `JWT_KEY` por variável de ambiente). O valor atual deve ser considerado **comprometido** | **rotacionar** para variável de ambiente no servidor — independente desta instalação. O `appsettings.json` **do repositório** está correto (só o placeholder): o problema é a cópia publicada, que o `deploy.ps1` preserva |

