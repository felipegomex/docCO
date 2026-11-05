---
id: 20-19-gateway-openwa-canal-de-whatsapp.09
titulo: "19.8 Chave da API do OpenWA com valor vazado no log"
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
tokens_estimados: 455
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.8 Chave da API do OpenWA com valor vazado no log

**Sintoma.** O valor de uma chave de API aparece no **log do terminal** (por exemplo, no log do primeiro boot ou na saÃ­da de um comando de criaÃ§Ã£o de chave) e passa a ser considerado **comprometido**.

**Causa.** A chave ADMIN inicial **Ã© exibida uma vez**, no log do primeiro boot. AlÃ©m disso, o `docker compose logs` imprime a linha de criaÃ§Ã£o da chave. O valor que aparece em log nÃ£o pode mais ser usado.

**SoluÃ§Ã£o â€” apagar e recriar, nÃ£o â€œreusar com outro nomeâ€:**

1. **Apague** a chave exposta (rota de gestÃ£o de chaves do gateway; a chave ADMIN inicial tambÃ©m pode ser revogada).
2. **Crie** uma chave nova com `POST /api/auth/api-keys`, com o corpo `{"name":"central-operacao","role":"operator"}` â€” a Central usa uma chave de escopo **amplo** (`operator`), porque cria a sessÃ£o dinamicamente ao cadastrar o nÃºmero (uma chave por sessÃ£o exigiria manutenÃ§Ã£o a cada cadastro).
3. **Atualize** `WhatsApp__ApiKey` no `web.config` do backend (Â§13.1) com o valor novo e **recycle** o backend.
4. **Confirme** a troca comparando a impressÃ£o digital (SHA-256, 16 caracteres) do valor no servidor e do valor no `.env` da VM â€” sem imprimir nenhum dos dois.
5. **Apague** o histÃ³rico do terminal/log que contÃ©m o valor antigo.

> NÃ£o hÃ¡ "trocar a chave mantendo o valor": chave exposta em log Ã© chave queimada. E o `docker compose logs` continua guardando o valor antigo â€” trate o log como parte do segredo.

