---
id: sec.07-6-pipeline-obrigatorio
titulo: "6. Pipeline obrigatório"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 3084
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 6. Pipeline obrigatório

### Etapa 0 — congelar o ponto de partida

Na raiz do monorepo:

```powershell
git branch --show-current
git status --short
git rev-parse HEAD
```

Condições:

- branch esperada, normalmente `developer`;
- `git status --short` sem saída;
- dependências instaladas;
- migrations que serão publicadas já identificadas;
- nenhum processo de build local capaz de alterar `dist`, `publish_output` ou `deploy`.

> ⚠️ **Deploy de árvore não commitada (2026-10-27).** O publish de HML e PROD foi feito a partir da **árvore de trabalho** sobre o HEAD `b52d7bc`, com mudanças de código e de schema ainda **sem commit**. O `BUILD_INFO.GitCommit` registrou o HEAD, mas os binários publicados **não correspondem** a ele. Na época o `preflight-schema.ps1` **não** verificava as colunas novas do RBAC (`CARGO_Nivel`/`CARGO_FuncaoId`) — hoje verifica —, então o publish passou e o **login em PROD respondeu 500** até aplicar `Permissoes_Idempotente.sql` + `Permissoes_Matriz_Idempotente.sql` (§13). Use essa modalidade só em emergência consciente, registre a divergência e aplique o SQL do commit; o caminho normal continua sendo commit → validate → SQL → publish.

### Etapa 1 — subir interno

Use a skill `subir interno` ou dois terminais.

Backend:

```powershell
Set-Location .\backend\Central_BackEnd
dotnet run
```

Frontend:

```powershell
Set-Location .\frontend
npm start
```

Teste em `http://localhost:4200`:

1. login;
2. navegação autenticada;
3. recarregamento/refresh da sessão;
4. ao menos um módulo afetado;
5. integração externa quando o fluxo exigir.

Não use build de produção nessa etapa.

### Etapa 2 — primeiro validate

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
```

O gate executa, nesta ordem:

1. `1/7` — `scripts/check-secrets.ps1` no índice do Git (**bloqueante**);
2. `2/7` — `git status`, branch e commit (aviso quando a árvore está suja);
3. `3/7` — `dotnet build --nologo` em `backend/Central_BackEnd` — **Debug**, pois não informa `-c Release`;
4. `4/7` — `npx ng build` em `frontend`;
5. `5/7` — `dotnet test --nologo -v q` em `backend/Central_BackEnd.Tests`;
6. `6/7` — `ng test --watch=false --browsers=ChromeHeadlessNoSandbox` no `frontend`;
7. `7/7` — `scripts/check-docs.ps1` (**bloqueante**).

Depois do passo 7, opcional, **em modo aviso** (não bloqueia): sync de docs em dry-run — dispara somente se `DOCS_SYNC_USUARIO` **e** `DOCS_SYNC_SENHA` existirem como variáveis de ambiente (`DOCS_SYNC_SERVIDOR`/`DOCS_SYNC_BANCO` são opcionais; veja [§ 7 Migrations](#7-migrations-antes-da-publicação) e o item do `TbDocDoc` na tabela abaixo).

⚠️ **O gate roda as duas suítes: backend (5/7) e frontend (6/7).** `dotnet test` cobre, entre outras, `ValidadoresAcesso`, `Cifrador`, a regressão de `TemSenha`, as regras de função/bloqueio/quadro do Kanban Geral e as suítes do espelho de documentação (`TbDocSchemaTests`, `DocumentosServiceTests`). No frontend, `ng test` executa em modo headless (`ChromeHeadlessNoSandbox`, passo 6/7); há dezenas de arquivos `*.spec.ts` (inventário de **177 casos** em 2026-10-27, inclusive guard de Conhecimento e os specs do WhatsApp) e o `test` target existe no `angular.json`; `frontend/karma.conf.js` existe.

O validate prova a compilação Debug, não os mesmos binários que serão publicados. O `dotnet publish -c Release` da etapa de package é o gate real do backend publicado. Se o validate retornar código 1, pare; se o package Release falhar, também pare e não publique.

### Etapa 3 — commit e push pelo fluxo normal

```powershell
git status --short
git add <arquivos-da-mudanca>
git diff --cached --stat
git commit -m "tipo: descricao"
git push origin developer
git status --short
git rev-parse HEAD
```

Use a convenção do repositório: `feat:`, `fix:`, `docs:`, `chore:` ou `refactor:`. Não inclua segredos, saída de build, `deploy/` ou `publish_output/`.

### Etapa 4 — segundo validate

Execute novamente:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
```

Esse segundo gate deve rodar com o commit já gravado. Se a árvore estiver suja ou o hash mudou, pare e corrija antes de empacotar.

### Etapa 5 — package sem tocar no IIS

Como `BuildFrontend` e `BuildBackend` são `Nullable[bool]`, use `powershell -Command` e escape o `$` no PowerShell externo:

```powershell
powershell -ExecutionPolicy Bypass -Command "& '.\scripts\deploy\deploy.ps1' -Ambiente HML -BuildFrontend `$true -BuildBackend `$true -Publicar `$false -Backup `$false"
```

O package:

- roda `npm run build` em `frontend`;
- roda `dotnet publish -c Release -o ./publish_output` no backend;
- valida os artefatos;
- copia com `robocopy /E` para `deploy/backend` e `deploy/frontend` — `/E` **não espelha nem limpa** arquivos antigos;
- grava `deploy/BUILD_INFO.txt` (com `Ambiente`, `Servidor` e destinos) e um recibo em `deploy/history/<AMBIENTE>-package-<timestamp>.txt` (pasta ignorada pelo Git);
- não acessa nem altera o IIS.

> Em `PROD`, a trava (`-Backup 1` + digitar `PROD`) é avaliada **antes** de qualquer build — empacotar com `-Ambiente PROD` sem `-Backup 1` aborta de imediato. Para gerar o pacote compartilhado sem acionar a trava, empacote com `-Ambiente HML`.

### Credenciais locais — `deploy-env.ps1`

Para não repetir `-UsuarioRemoto`/`-SenhaRemota` e `PREFLIGHT_DB_*` a cada publish, use o wrapper versionado [`scripts/deploy/deploy-env.ps1`](../../../scripts/deploy/deploy-env.ps1):

```powershell
# HML: empacotar e publicar (backup obrigatório em PROD)
.\scripts\deploy\deploy-env.ps1 -Ambiente HML -BuildFrontend 1 -BuildBackend 1 -Publicar 0
.\scripts\deploy\deploy-env.ps1 -Ambiente HML -Publicar 1 -Backup 1

# PROD
.\scripts\deploy\deploy-env.ps1 -Ambiente PROD -Publicar 1 -Backup 1 -ConfirmarProd
```

As credenciais ficam em `.env.local` (**gitignored**), com estas chaves:

```text
DEPLOY_HML_USUARIO / DEPLOY_HML_SENHA
DEPLOY_PROD_USUARIO / DEPLOY_PROD_SENHA
PREFLIGHT_DB_SERVIDOR / PREFLIGHT_DB_USUARIO / PREFLIGHT_DB_SENHA
PREFLIGHT_DB_BANCO_HML / PREFLIGHT_DB_BANCO_PROD
```

O wrapper valida as chaves, monta a `SecureString` e define `PREFLIGHT_DB_*` apenas no processo filho; nenhuma senha é impressa. **Nunca** commite `.env.local` nem `deploy.local.ps1`; restrinja o ACL do arquivo ao seu usuário.

Se houver risco de saída antiga, pare o frontend/backend local e limpe somente diretórios gerados antes de repetir o package:

```powershell
Remove-Item -Recurse -Force .\deploy -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force .\backend\Central_BackEnd\publish_output -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force .\frontend\dist\central-conhecimento-actyon -ErrorAction SilentlyContinue
```

Depois execute o package completo novamente; não apague código-fonte.

Confira:

```powershell
git status --short
git rev-parse HEAD
Get-Content .\deploy\BUILD_INFO.txt
Test-Path .\deploy\backend\Central_BackEnd.dll
Test-Path .\deploy\backend\Central_BackEnd.runtimeconfig.json
Test-Path .\deploy\frontend\browser\index.html
```

`BUILD_INFO.GitCommit` deve ser igual a `git rev-parse HEAD`.

### Etapa 6 — publish sem rebuild

Não passe pela linha de comando do processo filho para credencial. No mesmo PowerShell, obtenha a credencial segura e invoque o script:

```powershell
$usuarioIis = Read-Host "Usuário administrativo do IIS"
$credencialIis = Get-Credential -UserName $usuarioIis -Message "Credencial de publicação IIS"

try {
    & '.\scripts\deploy\deploy.ps1' `
        -Ambiente HML `
        -BuildFrontend $false `
        -BuildBackend $false `
        -Publicar $true `
        -Backup $true `
        -UsuarioRemoto $credencialIis.UserName `
        -SenhaRemota $credencialIis.Password
    # Para PROD, troque por -Ambiente PROD (exige -Backup $true e digitar PROD na trava;
    # só publique em PROD após smoke verde em HML, sem rebuild entre os dois).
}
finally {
    $credencialIis = $null
    $usuarioIis = $null
}
```

A publicação:

1. em `PROD`, exige `-Backup $true` e confirmação digitando `PROD` (aborta antes de tocar no IIS);
2. compara `BUILD_INFO.GitCommit` com o HEAD antes de tocar o servidor;
3. testa conectividade;
4. autentica no compartilhamento administrativo (sem `-SenhaRemota` explícito, aborta);
5. executa o pre-flight no servidor (fase 5b, **antes** do `app_offline.htm`): aborta se houver `app_offline.htm` residual, se faltar o `appsettings.json` do ambiente ou o `runtime-config.json` do frontend;
6. cria backup dos sites;
7. publica backend com `app_offline.htm`;
8. remove o arquivo de offline após backend íntegro;
9. publica o frontend preservando o `runtime-config.json` do servidor;
10. encerra a unidade de rede remota e grava o recibo em `deploy/history/<AMBIENTE>-publish-<timestamp>.txt`.

Se ocorrer exceção, erro do PowerShell ou código diferente de zero, não rode smoke como se o deploy tivesse passado. Consulte o terminal e o estado `app_offline.htm`.

> ⚠️ **Depois do publish: reaplicar o bloco `<environmentVariables>` do `web.config` do backend.** O `deploy.ps1` espelha o backend com `/MIR` excluindo **apenas** `appsettings*.json` e `app_offline.htm`: o `web.config` publicado é **sobrescrito** e o bloco manual (`WhatsApp__Habilitado`, `WhatsApp__BaseUrl`, `WhatsApp__WebhookUrl`, `WhatsApp__ApiKey`, `WhatsApp__WebhookSecret`) **segue junto** — o script **não** o preserva. Com o módulo WhatsApp ativo, o bloco precisa ser **reaplicado depois de cada deploy** (e o pool reciclado). O bloco tem que ser **filho de `<aspNetCore>`** e usar `<environmentVariable name= value=>`; o schema de `applicationHost.config` (`<add key= value=/>`) **derruba o site inteiro** com `500` de corpo vazio em todas as rotas — inclusive em rota inexistente, que deveria ser `404`. O **detector** desse problema é o `smoke.ps1`: se o frontend passa e o check **"Swagger/API"** falha, o backend não subiu. Detalhe em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §7, [`troubleshooting.md`](../troubleshooting.md) §13.1 e §19.9.5.
>
> ⚠️ **E reaplicar o script de schema quando houver mudança de largura.** O mesmo deploy do módulo WhatsApp pode alterar o **script de banco** (`Migrations/Sql/WhatsApp_Idempotente.sql`): o caso real foi o alargamento de `tbwhatsapp_numero.WAT_Status` de `varchar(20)` para **`varchar(50)`**, com o **bloco idempotente de ajuste** que conserta base já instalada. O `deploy.ps1` **não** aplica DDL: o script vai **antes** do publish, e o `deploy.ps1` sozinho deixa o código novo contra o schema velho — o SQL Server volta a devolver o **erro 8152** ("dados seriam truncados"), a linha da fila fica presa em "processando" e o telefone do número nunca é gravado.

### Etapa 7 — smoke real

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1 -Servidor 192.168.2.130  # HML
powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1 -Servidor 192.168.2.254  # PROD (IIS PROD)
```

O script atual testa quatro checks (todos obrigatórios para saída 0):

- frontend na porta 1010;
- Swagger na porta 1009 (`/swagger/v1/swagger.json` com `openapi` e `paths` não vazio);
- `runtime-config.json` do frontend servido, com `apiBaseUrl` igual à do ambiente (`http://<servidor>:1009/api/v1`);
- **login fictício** em `POST /api/v1/auth/login`: o esperado é **401**; **500** indica schema/código quebrado (foi a causa do incidente de 2026-10-21 com `CARGO_ID`).

Ele não faz login válido e não chama API de negócio. **O check "Swagger/API" é o detector do `web.config` quebrado**: se o frontend passa e só o Swagger falha, o backend **não subiu** — quase sempre é o bloco `<environmentVariables>` com o schema errado, que precisa ser reaplicado depois do publish (ver o aviso da Etapa 6 e [`troubleshooting.md`](../troubleshooting.md) §13.1). Após o smoke, faça um gate manual mínimo:

- abrir a aplicação;
- autenticar;
- recarregar e renovar sessão;
- abrir uma tela autenticada e verificar o logout, registrando apenas o status;
- consultar ao menos um endpoint de negócio representativo;
- verificar a funcionalidade afetada.

Consulte [`09-TROUBLESHOOTING.md`](../troubleshooting.md) para diagnóstico.

