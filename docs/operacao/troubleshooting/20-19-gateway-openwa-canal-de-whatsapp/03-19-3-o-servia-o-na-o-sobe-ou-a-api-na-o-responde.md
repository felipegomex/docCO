---
id: 20-19-gateway-openwa-canal-de-whatsapp.03
titulo: "19.3 O serviÃ§o nÃ£o sobe ou a API nÃ£o responde"
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
tokens_estimados: 534
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.3 O serviÃ§o nÃ£o sobe ou a API nÃ£o responde

| Sintoma | Causa | SoluÃ§Ã£o |
|---|---|---|
| `systemctl status openwa` parado (caminho nativo) ou `docker compose ps` mostra o container `openwa-api` fora (caminho Docker) | O serviÃ§o nÃ£o subiu, ou o binÃ¡rio falhou ao subir | Confira o log (`journalctl -u openwa` ou `docker compose logs --tail=100 openwa-api`); no caminho nativo, confirme o **diretÃ³rio de trabalho**, o `EnvironmentFile` e o runtime de produÃ§Ã£o (`node dist/main`, nunca `npm run dev`); o serviÃ§o precisa de **inicializaÃ§Ã£o automÃ¡tica** e **reinÃ­cio em falha** |
| `GET http://192.168.2.253:2785/api/health` devolve `401`/`403` | Falta o header `X-API-Key` ou a chave nÃ£o confere | A chave Ã© a **mesma** do backend (`WhatsApp__ApiKey`). Compare a impressÃ£o digital (SHA-256) dos dois lados em vez de imprimir o valor |
| O gateway responde em `127.0.0.1` mas os servidores da Central **nÃ£o** o alcanÃ§am | No caminho Docker, o `docker-compose.yml` publica a porta **apenas em loopback** (`127.0.0.1:${API_PORT:-2785}:2785`); ou o firewall da VM nÃ£o libera a origem | Publique a porta no IP da VM com um `docker-compose.override.yml` que substitua a lista `ports`, e libere `2785` na VM **apenas** para `192.168.2.130` e `192.168.2.254` |
| O backend responde `502` nas aÃ§Ãµes de envio | Falha na chamada ao gateway (rede, timeout de 30 s ou status nÃ£o-2xx) | O `502` diz "o outro lado falhou", nÃ£o "o pedido estÃ¡ errado". Olhe o log do backend: a rota e o status do gateway estÃ£o registrados, **sem** a chave. Verifique se o gateway estÃ¡ de pÃ© e se `WhatsApp:BaseUrl` aponta para `http://192.168.2.253:2785/api` â€” o `appsettings.json` versionado ainda traz `http://127.0.0.1:2785/api`, herdado do desenho antigo, e precisa ser sobrescrito por `WhatsApp__BaseUrl` |

