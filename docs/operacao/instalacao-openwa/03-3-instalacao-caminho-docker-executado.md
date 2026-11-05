---
id: instalacao-openwa.03
titulo: "3. Instalação (caminho Docker — executado)"
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
tokens_estimados: 1370
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 3. Instalação (caminho Docker — executado)

### 3.1 Código

```bash
git clone https://github.com/rmyndharis/OpenWA.git ~/openwa
cd ~/openwa
git checkout v0.25.0
```

A versão está **fixada** na tag `v0.25.0`: não acompanhe `latest`. Uma atualização é operação de manutenção (§10).

### 3.2 `docker-compose.override.yml` — os ajustes efetivos

O `docker-compose.yml` do projeto **constrói** a imagem a partir do código (`build: context: .`) e publica a porta **apenas em loopback** (`127.0.0.1:${API_PORT:-2785}:2785`). O `override` no mesmo diretório corrige isso e fixa os limites de recurso do container:

```yaml
services:
  openwa-api:
    image: ghcr.io/rmyndharis/openwa:0.25.0   # imagem oficial fixada no lugar do build
    ports: !override                            # sem !override, as portas se somariam
      - '192.168.2.253:2785:2785'               # publica no IP da VM, não em 0.0.0.0
    shm_size: 2gb                               # memória compartilhada do Chromium
    mem_limit: 4g                               # teto de memória do container
    pids_limit: 2048                            # teto de processos
  docker-proxy:
    profiles: ['disabled']                      # fora de qualquer perfil => não sobe
```

Por que cada ajuste:

| Ajuste | Motivo |
|---|---|
| `image:` no lugar de `build:` | não compila o projeto na VM; a versão vem da imagem oficial do GHCR, fixada em `0.25.0` |
| `ports: !override` | o compose oficial publica **só em loopback**; sem o `!override` a lista seria somada, não substituída. Vincular ao IP da VM (e não a `0.0.0.0`) mantém o firewall como segunda camada |
| `shm_size: 2gb` | o Chromium do `whatsapp-web.js` usa `/dev/shm` (memória compartilhada) e o **default do Docker é 64 MB**: com esse valor a aba da sessão **quebra** — *crash de página*, a sessão cai sozinha e **sem erro no log**. Com 2 GB a sessão fica de pé |
| `mem_limit: 4g` | teto de memória do container (o `whatsapp-web.js` roda um Chromium por sessão: ~300–500 MB) |
| `pids_limit: 2048` | teto de processos — o Chromium abre vários por sessão, e o default do host pode ser baixo demais |
| `docker-proxy` com `profiles: ['disabled']` | não usamos a orquestração de datastores embutidos; com o profile desativado **nenhum** container ganha acesso ao socket do Docker. O `depends_on: docker-proxy` do compose oficial é `required: false`, então desativar o proxy **não** quebra o `up` |

O **`override` não troca o volume**: o estado (perfil de sessão, SQLite, mídia, chaves) continua no volume nomeado **`openwa_openwa-data`** — é ele que faz o número voltar **pareado**, sem novo QR (§10.1).

### 3.3 `.env` na própria VM

O `.env` (no mesmo diretório do compose) é lido pelo Compose e **fixa** o valor de cada chave que aparece nele. Crie-o **na VM**, com os segredos gerados nela, permissão `600` e **fora** de qualquer repositório versionado:

```dotenv
API_MASTER_KEY=SUA_CHAVE
API_KEY_PEPPER=SUA_CHAVE
SSRF_ALLOWED_HOSTS=192.168.2.130,192.168.2.254
CSP_UPGRADE_INSECURE_REQUESTS=false
ENABLE_SWAGGER=false
DATABASE_TYPE=sqlite
STORAGE_TYPE=local
REDIS_ENABLED=false
QUEUE_ENABLED=false
ENGINE_TYPE=whatsapp-web.js
AUTO_START_SESSIONS=true
MAX_CONCURRENT_SESSIONS=2
OPENWA_MEM_LIMIT=4g
```

Geração dos segredos **na VM**, sem eco do valor:

```bash
openssl rand -base64 36   # API_MASTER_KEY
openssl rand -base64 36   # API_KEY_PEPPER
chmod 600 .env
```

| Chave | Por quê |
|---|---|
| `API_MASTER_KEY` | semente da primeira chave ADMIN; o boot de produção recusa valor curto ou default conhecido. **Definir antes do primeiro boot** |
| `API_KEY_PEPPER` | pepper do hash das chaves de API (HMAC-SHA256). **Trocar depois bloqueia todas as chaves**, inclusive a ADMIN |
| `SSRF_ALLOWED_HOSTS` | **obrigatório**: o guard de SSRF do gateway recusa **registrar** webhook para endereço privado sem essa lista. Sem ela, o `POST /api/sessions/{id}/webhooks` com a URL do backend é recusado |
| `CSP_UPGRADE_INSECURE_REQUESTS=false` | o dashboard é servido em HTTP puro; sem isto o navegador sobe os scripts para `https` e a tela fica **branca** |
| `ENABLE_SWAGGER=false` | o Swagger não é exposto no ambiente de produção |
| `DATABASE_TYPE=sqlite` + `STORAGE_TYPE=local` | perfil mínimo: sem PostgreSQL, sem MinIO, sem Redis e sem fila |
| `AUTO_START_SESSIONS=true` | reconecta as sessões já autenticadas após o boot; sem isso a sessão fica `disconnected` e precisa de `POST /api/sessions/{id}/start` |
| `MAX_CONCURRENT_SESSIONS=2` | teto de sessões simultâneas (cada uma é um Chromium) |
| `OPENWA_MEM_LIMIT=4g` | limite de memória do container |

### 3.4 Subir e verificar

```bash
cd ~/openwa
docker compose up -d
docker compose ps
docker compose logs --tail=100 openwa-api
```

O serviço fica `restart: unless-stopped`: o container **sobe sozinho** com o daemon do Docker, mas o gateway **não** aplica migrations no boot e a VM **não** executa `docker compose up` automaticamente. Todo o estado (SQLite, sessões, mídia, chaves) vive no volume nomeado `openwa_openwa-data`.

Verificação de saúde (a chave é a mesma do backend — ver §8):

```bash
curl -s -H "X-API-Key: SUA_CHAVE" http://127.0.0.1:2785/api/health
```

