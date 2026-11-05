---
id: sec.11-10-preservacao-de-configuracao
titulo: "10. Preservação de configuração"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - scripts/deploy
  - backend/Central_BackEnd/Services/Configuracao
testes:
  - docs/
tokens_estimados: 2900
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 10. Preservação de configuração

### Preservado

O mirror do backend exclui `appsettings*.json` (`/XF`), incluindo o arquivo verificado após a publicação. Isso mantém a string de conexão, JWT e demais ajustes do servidor. **Fato atual:** o pre-flight **aborta** o publish se o `appsettings.json` do ambiente faltar no servidor — ausência de configuração é hard stop, não aviso.

O `runtime-config.json` do frontend (`assets\config\runtime-config.json`) também é preservado: a fase 8 lê o arquivo do servidor antes do mirror e o restaura depois. Mas se ele **não existir** no servidor (primeiro deploy), o pre-flight aborta — crie o arquivo no servidor com a `apiBaseUrl` do ambiente antes do publish, senão o front cai no `localhost` do pacote.

**Fase 09 — `whatsappHubUrl` no `runtime-config.json`.** O campo é opcional. Desde a rodada de 2026-10-30, o frontend **só aceita** o valor quando a origem é a **mesma** da `apiBaseUrl` e não é `localhost`/`127.0.0.1` num host remoto; caso contrário deriva `{origem da apiBaseUrl}/hubs/whatsapp`. Assim, um `runtime-config.json` preservado de HML (ou o `localhost` do arquivo versionado) **não** arrasta mais o hub no deploy de PROD (`.254`): o hub acompanha a API do ambiente. Ainda assim, manter o `whatsappHubUrl` explícito no servidor é o caminho recomendado para leitura; o pre-flight/smoke atuais **não** validam esse campo, então confira a conexão do hub no navegador após publicar.

Ao publicar o módulo Acessos Empresas, confirme que o `appsettings*.json` do servidor **mantém a seção `Acessos`**. Desde a implementação de segredos no banco (2026-10-30), `Acessos:ChaveCifra` deixou de ser obrigatória no Chat/Conhecimento: o placeholder é tratado como ausente e a chave-mestra é gerada e gravada em `tbconfiguracao_sistema` (ver a subseção abaixo). A variável continua válida como **override** e, quando presente e base64 de 32 bytes, tem precedência sobre o banco. Confirme também a cobertura do backfill (linhas sem o prefixo `v1:`).

### Preservado condicionalmente

- `appsettings*.json` é **sempre** preservado: sai do espelho do mirror e o
  pre-flight aborta o publish se o arquivo faltar no servidor.
- `web.config` do backend é preservado **só quando** contém variáveis
  `WhatsApp__*`: nesse caso `scripts/deploy/deploy.ps1` tira o arquivo do
  espelho e grava uma cópia datada `web.config.pre-deploy-<timestamp>`. **Sem**
  esse marcador o publish sobrescreve o arquivo e o script avisa no log.

### Não preservado como configuração manual

- ⚠️ Consequência prática: o bloco `<environmentVariables>` do `web.config` do backend (onde entram `WhatsApp__Habilitado`, `WhatsApp__BaseUrl`, `WhatsApp__WebhookUrl`, `WhatsApp__ApiKey` e `WhatsApp__WebhookSecret`) **só sobrevive ao publish enquanto o arquivo tiver o marcador `WhatsApp__*`** — um servidor configurado apenas com `AnythingLLM__*`/`Gitea__*` perderia o `web.config` no próximo deploy. Quando o bloco é perdido, ele precisa ser **reaplicado no servidor** e o backend **reciclado**. Sem isso, o canal de WhatsApp cai sem erro visível. Procedimento e sintoma em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §7 e [`troubleshooting.md`](../troubleshooting.md) §13.1. **Estado em 2026-10-28:** os dois servidores (PROD `192.168.2.254` e HML `192.168.2.130`) já tinham `WhatsApp__*` no `web.config` — por isso o arquivo **foi preservado** no publish e o bloco do chat (`AnythingLLM__*`/`Gitea__*`/`KnowledgeSync__*`) sobreviveu; o publish gravou a cópia datada `web.config.pre-deploy-<timestamp>`.
- ⚠️ O **mesmo deploy** pode trazer mudança no **script de schema** do módulo (`Migrations/Sql/WhatsApp_Idempotente.sql`): o caso real foi o alargamento de `tbwhatsapp_numero.WAT_Status` para **`varchar(50)`**, com o **bloco idempotente de ajuste** que conserta base já instalada. O script **precisa ser reaplicado no banco do ambiente** quando houver mudança de largura — e **antes** do publish (o `deploy.ps1` não aplica DDL). Sem isso, o código novo roda contra o schema velho e o SQL Server volta a devolver o **erro 8152** ("dados seriam truncados"), com a linha da fila presa em "processando" e o telefone do número nunca gravado. Detalhe em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §6.3 e [`troubleshooting.md`](../troubleshooting.md) §19.10.1.
- Variáveis de ambiente do Application Pool não fazem parte do pacote.
- Segredos devem ficar em armazenamento seguro/variáveis, não em `web.config` versionado.

### Chat/Conhecimento — env vars do servidor (fonte da verdade)

A configuração do Chat/Conhecimento **não** vem do `appsettings.json` versionado
(que fica com `AnythingLLM:ApiKey` **vazio** e **sem** as seções `Gitea` e
`KnowledgeSync`). A fonte da verdade é o **`web.config` do backend no servidor**,
no bloco `<environmentVariables>`:

| Ambiente | Servidor | Caminho do `web.config` |
|---|---|---|
| PROD | `192.168.2.254` | `inetpub\wwwroot\CentralOperacao\Back\web.config` |
| HML | `192.168.2.130` | `inetpub\wwwroot\Suporte_Back\web.config` |

Variáveis usadas pelo Chat/Conhecimento: `AnythingLLM__BaseUrl`,
`AnythingLLM__ApiKey`, `AnythingLLM__TimeoutSeconds`,
`AnythingLLM__StatusTimeoutSeconds`, `Gitea__BaseUrl`, `Gitea__Owner`,
`Gitea__Repository`, `Gitea__MainBranch`, `Gitea__OAuth__AuthorizeUrl`,
`Gitea__OAuth__TokenUrl`, `Gitea__OAuth__ClientId`, `Gitea__OAuth__ClientSecret`,
`Gitea__OAuth__CallbackUrl`, `Gitea__OAuth__FrontendUrl` e
`KnowledgeSync__TriggerUrl`.

Regras práticas:

- **Alterar o `web.config` reinicia a aplicação no IIS** (não é deploy): faça
  backup datado antes (ex.: `web.config.bak-20261028-184627`).
- **Nenhum valor de segredo no repositório**: `client_id`/`client_secret` do
  Gitea e a chave do AnythingLLM ficam **fora do repo** (`.opencode/secrets/`,
  gitignored) e **só** no servidor.
- Estado verificado (2026-10-28): **PROD e HML configurados** com o mesmo bloco
  de env vars (o chat responde no iframe do AnythingLLM via SSO; `POST
  /api/conhecimento/gitea/oauth/iniciar` → 200, `GET /api/conhecimento/permissao`
  → 200 **nos dois**). SQL do chat aplicado em 2026-10-30.
- O `runtime-config.json` do front é a mesma ideia no lado do Angular: no PROD
  ele aponta para `http://192.168.2.254:1009/api/v1`; o `192.168.2.130:1009`
  que aparece no bundle é só o fallback de
  `frontend/src/environments/environment.prod.ts`.

### Chat/Conhecimento — segredos no banco e chaveiro do DataProtection (2026-10-30)

A partir desta implementação (código pronto; **SQL aplicado em HML/PROD em 2026-10-30**, sem deploy),
`Acessos__ChaveCifra` e `Gitea__ServiceToken` deixam de ser obrigatórias para o
Chat/Conhecimento:

- **Chave-mestra automática:** na primeira subida, se `Acessos:ChaveCifra` não
  for um base64 válido de 32 bytes (o placeholder do `appsettings.json` é
  tratado como ausente), a chave é gerada e gravada em `tbconfiguracao_sistema`
  (`acessos.chave_cifra`), **cifrada pelo DataProtection**
  (`AddDataProtection().SetApplicationName("Central.Conhecimento")`,
  `PersistKeysToFileSystem`).
- **Override:** se `Acessos:ChaveCifra` existir em `appsettings`/env e for válida,
  ela tem **precedência** e nada é criado no banco.
- **Token do Gitea:** gravado cifrado (AES-GCM) em `tbconfiguracao_sistema`
  (`gitea.service_token`) pela tela admin `/conhecimento/configuracao`; a
  variável `Gitea__ServiceToken` continua como fallback. `503` na aprovação só
  quando não há token em nenhuma das duas origens.

**Chaveiro do DataProtection — backup obrigatório.** A chave que decifra a
chave-mestra fica fora do banco, neste caminho (o primeiro que existir):

1. `CENTRAL_DATAPROTECTION_PATH` (variável de ambiente, quando definida);
2. `%ProgramData%\Central\DataProtection-Keys`;
3. `App_Data/DataProtection-Keys` (fallback relativo ao site).

Perder esse diretório torna a chave-mestra gravada no banco **ilegível** — e o
código **não sobrescreve** uma chave ilegível. Antes de publicar ou de trocar de
servidor, faça **backup do diretório do chaveiro** junto com o backup do IIS.

### `web.config` do frontend — vem do repo (2026-10-28)

O `web.config` do **front** é **do repositório** (`frontend/public/web.config`):
o build o copia para `browser/web.config` e o `scripts/deploy/deploy.ps1` o
publica na raiz do site **a cada deploy**. Ele **não** é configuração manual
preservada — quem manda é o repo, e o publish sobrescreve o arquivo do
servidor.

**Estado atual:** o arquivo **não** tem nenhuma regra que exclua `/api/*` do
fallback SPA. A tentativa de 2026-10-28 (commit `7ac8c61`, condição com
`matchType="RegexPattern"`) **derrubou os dois sites** — o módulo URL Rewrite
do IIS não inicializou e todas as requisições responderam **500**
(`500/50/13` no log) — e foi **revertida** em `a5e2df6`. Hoje o site do front
responde **200 com o `index.html`** em `/api/*`: isso **não** é defeito, porque
o chat e o conhecimento chamam a API na **origem da runtime-config** (`:1009`)
via `ConfigService#apiRootUrl` e nunca chamam `/api/*` no próprio origin. O
conserto do chat **não depende** do `web.config`.

Implicação no deploy:

1. O `web.config` do front **vem do repo** — qualquer ajuste de regra é feito em
   `frontend/public/web.config` e só vale **após o próximo publish**.
2. **Não** copie o `web.config` do front de um ambiente para outro: PROD e HML
   usam o **mesmo** arquivo do repo.
3. O `web.config` do **backend** segue a **regra própria** (preservado só
   quando tem `WhatsApp__*`) — são arquivos diferentes, com regras diferentes.
4. **Antes** de publicar qualquer mudança no `web.config` do front, valide o
   XML e o comportamento nos **dois** ambientes: uma condição inválida no URL
   Rewrite derruba o site inteiro (ver o procedimento abaixo).

### Editar `web.config` no servidor com segurança (front e back)

1. **Copie o arquivo do servidor para a estação** em cópia **binária**
   (SMB/`Copy-Item`), sem converter linha de código nem codificação.
2. **Edite localmente** e **valide o XML** antes de devolver
   (ex.: `[xml](Get-Content -Raw <arquivo>)`).
3. **Copie de volta preservando bytes** e recycle do app pool (alterar o
   `web.config` reinicia a aplicação no IIS).
4. **Nunca** use o redirecionamento `>` do **PowerShell** para escrever
   arquivo de config do IIS: ele grava **UTF-16** e o IIS passa a responder
   **500.19** (configuração ilegível).
5. Ao restaurar do Git, use **bytes crus**:
   `cmd /c "git show <commit>:frontend/public/web.config > arquivo"` — o `cmd`
   preserva o UTF-8 sem BOM; o `>` do PowerShell não.
6. **Valide** com `scripts/smoke.ps1` (front **200** e `runtime-config.json`
   **200**); no incidente de `500/50/13`, confira o log do IIS.

### Publicação de 2026-10-28 (HML e PROD) — o que foi preservado

| Item | HML (`192.168.2.130`) | PROD (`192.168.2.254`) |
|---|---|---|
| `web.config` preservado no publish | sim — o arquivo tinha `WhatsApp__*` | sim — o arquivo tinha `WhatsApp__*` |
| Cópia datada do publish | `web.config.pre-deploy-20261028-192811` | `web.config.pre-deploy-20261028-192855` |
| Backup antes de gravar o bloco do chat | `web.config.bak-20261028-192936` | `web.config.bak-20261028-184627` |
| Backup do IIS | `\\192.168.2.130\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261028-192800` | `\\192.168.2.254\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261028-192848` |
| Publicações seguintes (2026-10-28) | `7ac8c61` — backup IIS `20261028-201714`, preflight OK, smoke OK; depois o `web.config` derrubou o site (`500.50.13`) | `7ac8c61` — backup IIS `20261028-201735`, preflight OK, smoke OK; depois o `web.config` derrubou o site (`500.50.13`) |
| `appsettings.json` do servidor | preservado, **sem segredo** (placeholder vazio) | preservado, **sem segredo** (placeholder vazio) |
| Env vars do chat | mesmo bloco do PROD (20 no total) | 20 no total |

Ordem usada: (1) publish com backup — o `web.config` sai do espelho porque tem
`WhatsApp__*`; (2) gravação do bloco de env vars do chat **via SMB** com a conta
de serviço do deploy (`JCASRV-SUP`), com **backup datado** antes; (3) recycle do
app pool (alterar o `web.config` reinicia a aplicação no IIS); (4) smoke por
ambiente: `scripts/smoke.ps1 -Servidor 192.168.2.130` e
`scripts/smoke.ps1 -Servidor 192.168.2.254`.

### Configuração efetiva

`appsettings*.json` do servidor tem precedência somente quando não há override de ambiente. Para Swagger, `ASPNETCORE_SWAGGER_ENABLED` sobrepõe `SwaggerEnabled`.

Atenção ao JWT: desde `57a7e4f` a chave tem **fonte única** — `JwtKeyProvider` (singleton) resolve `JWT_KEY ?? Jwt:Key` uma vez, e emissão (`AuthService.GerarJwt`) e validação (`Program.cs`) usam a mesma chave. O que ainda é manual: disponibilizar `JWT_KEY` por canal seguro no ambiente da **máquina** do IIS (no *application pool* a recyclable apaga a chave e derruba as sessões) e validar login/refresh após a reciclagem. O startup loga `JWT: chave resolvida de {Origem}` e avisa quando a chave efetiva é placeholder — nesse caso qualquer instância valida tokens alheios e o publish deve ser interrompido. O cookie de refresh também está configurado no código com `Secure=false`; não trate HTTP como transporte seguro. Antes de HTTPS, alinhe CORS, bindings, URL do frontend e atributo `Secure`, depois repita login/refresh/logout.

