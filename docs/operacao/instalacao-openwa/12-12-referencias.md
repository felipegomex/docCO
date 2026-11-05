---
id: instalacao-openwa.12
titulo: "12. Referências"
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
tokens_estimados: 239
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 12. Referências

- [`README.md`](../../../material-de-projeto/atendimento-whatsapp/README.md) — documentação única do módulo: arquitetura, fluxo, regras, schema, API, tempo real e operação.
- [`troubleshooting.md`](../troubleshooting.md) — diagnóstico do canal (webhook 401/503, fila de webhook, `web.config` do IIS, firewall do Docker, chave de API vazada).
- [`deploy.md`](../deploy.md) — publicação da Central e procedimento de schema (o gateway **não** faz parte do deploy do IIS).
- [`scripts/infra/habilitar-openssh-chave.ps1`](../../../scripts/infra/habilitar-openssh-chave.ps1) — OpenSSH por chave pública no servidor IIS.
