---
id: 20-19-gateway-openwa-canal-de-whatsapp.02
titulo: "19.2 Todo envio falha com `the engine returned no message`: *patchers* do `whatsapp-web.js` nÃ£o aplicados"
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
tokens_estimados: 464
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.2 Todo envio falha com `the engine returned no message`: *patchers* do `whatsapp-web.js` nÃ£o aplicados

| Sintoma | Causa | SoluÃ§Ã£o |
|---|---|---|
| A sessÃ£o **conecta** (QR lido, status `ready`), mas **toda mensagem enviada pelo painel falha** com `the engine returned no message` | Os *patchers* do `whatsapp-web.js` **nÃ£o foram aplicados**. No caminho nativo isso acontece quando o `npm ci` rodou com `--ignore-scripts` (obrigatÃ³rio, para nÃ£o compilar o `better-sqlite3` v13) e os 12 *patchers* nÃ£o foram executados **na ordem do `Dockerfile`** â€” o hook `postinstall` que os aplica Ã© justamente um script desligado | Aplique os 12 *patchers* manualmente, na ordem do `Dockerfile` do OpenWA (o de status depois dos dois de que depende; o de send-error depois dos que editam `Client.js`). O aplicador preferencial Ã© o binÃ¡rio **GNU `patch`**; sÃ³ na ausÃªncia dele os *patchers* caem no **`git apply`** â€” por isso `git` precisa estar instalado. Um *patcher* que falha **aborta**: a Ã¡rvore meio corrigida nÃ£o se repara sozinha (`rm -rf node_modules/whatsapp-web.js && npm ci` e repetir). Reinicie o gateway e **teste um envio real** â€” Ã© o Ãºnico teste que prova o fim a end. âš ï¸ Este procedimento Ã© do caminho **nativo**: no caminho **Docker** (o instalado, com a imagem oficial `ghcr.io/rmyndharis/openwa:0.25.0` fixada) os *patchers* acompanham o build da imagem â€” se o mesmo sintoma aparecer, confirme a **versÃ£o da imagem** e o **log do container** antes de partir para a aplicaÃ§Ã£o manual |

