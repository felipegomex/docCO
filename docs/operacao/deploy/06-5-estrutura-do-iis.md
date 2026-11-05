---
id: sec.06-5-estrutura-do-iis
titulo: "5. Estrutura do IIS"
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
tokens_estimados: 1421
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 5. Estrutura do IIS

| `-Ambiente` | Camada | Binding | Pasta física | Conteúdo publicado |
|---|---|---|---|---|
| `HML` | Backend | porta 1009 | `C:\inetpub\wwwroot\Suporte_Back` | `deploy/backend` |
| `HML` | Frontend | porta 1010 | `C:\inetpub\wwwroot\Suporte_Front` | `deploy/frontend/browser` |
| `PROD` | Backend | porta 1009 | `C:\inetpub\wwwroot\CentralOperacao\Back` | `deploy/backend` |
| `PROD` | Frontend | porta 1010 | `C:\inetpub\wwwroot\CentralOperacao\FRONT` | `deploy/frontend/browser` |

### Backend

- Executa `Central_BackEnd.dll` pelo ASP.NET Core Module.
- `web.config` vem do `dotnet publish` e **é sobrescrito** no mirror.
- ⚠️ **Com o módulo WhatsApp ativo, o bloco `<environmentVariables>` do `web.config` do backend precisa ser reaplicado DEPOIS do publish.** O `deploy.ps1` espelha o backend com `/MIR` excluindo **apenas** `appsettings*.json` e `app_offline.htm`: o `web.config` publicado é sobrescrito e o bloco manual (`WhatsApp__Habilitado`, `WhatsApp__BaseUrl`, `WhatsApp__WebhookUrl`, `WhatsApp__ApiKey`, `WhatsApp__WebhookSecret`) **segue junto**. Sem reaplicar, o backend sobe sem as variáveis do canal. O bloco tem que ser **filho de `<aspNetCore>`** e usar `<environmentVariable name= value=>` — o schema de `applicationHost.config` (`<add key= value=/>`) **derruba o site inteiro** com `500` de corpo vazio em todas as rotas. Depois de editar, recicle: crie e remova `app_offline.htm` na pasta do backend ou recicle o application pool. Detalhe em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §7 e [`troubleshooting.md`](../troubleshooting.md) §13.1.
  - **Historicizado (2026-10-22):** o bloco ficou perdido no `.130` após um publish anterior e o módulo ficou desabilitado (webhook `503`). Foi **restaurado** a partir do backup `Backup_IIS\20261021-191450\backend\web.config` (mescla do bloco no `web.config` em uso, que voltou a ser `<aspNetCore ... />` self-closing após o publish validado do chat de agentes do mesmo dia) e os **deploy subsequentes passam a preservá-lo** (detector `WhatsApp__` em `deploy.ps1`, que guarda cópia datada ao lado). Smoke pós-restore: front/swagger/runtime **200**, login-fictício **401**, webhook **401 sem assinatura** (HMAC ativo = módulo no ar e segredo carregado). O `saude` responde **401 sem JWT** (endpoint autenticado; use token válido para conferir o estado do gateway).
- ⚠️ **O mesmo deploy do módulo WhatsApp tem uma segunda metade, do lado do banco.** Além do bloco do `web.config`, a publicação pode vir acompanhada de **mudança no script de schema** — por exemplo, o alargamento de `tbwhatsapp_numero.WAT_Status` para **`varchar(50)`** (o nome do evento `session.authenticated` tem 21 caracteres e não cabia em `varchar(20)`: o SQL Server devolvia o **erro 8152**, a linha da fila ficava presa em "processando" e o telefone do número nunca era gravado). O script `WhatsApp_Idempotente.sql` carrega um **bloco idempotente de ajuste de largura**, que alarga a coluna em base já instalada sem recriar a tabela. ⚠️ **O script precisa ser reaplicado no banco do ambiente quando houver mudança de largura** — e **antes** do publish: `deploy.ps1` não aplica DDL, e código novo contra schema antigo devolve o erro 8152 de novo. Detalhe em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §6.3 e [`troubleshooting.md`](../troubleshooting.md) §19.10.1.
- `appsettings*.json` do servidor são excluídos do mirror (`/XF`) e preservados; se `appsettings.json` faltar no servidor, o pre-flight **aborta** antes do `app_offline.htm`.
- `assets\config\runtime-config.json` do servidor é preservado (backup/restauração na fase 8); se faltar, o pre-flight **aborta** — no primeiro deploy, crie o arquivo no servidor com a `apiBaseUrl` do ambiente antes do publish.
- O script cria `app_offline.htm` antes de copiar e o remove somente após backend íntegro.
- O `web.config` versionado atualmente habilita Swagger e stdout log. Se a política for diferente, altere a fonte versionada/compatível e publique novamente; uma edição manual no servidor pode ser perdida.

### Frontend

- O build inclui `frontend/public/web.config` com fallback para `index.html`.
- Esse mesmo `web.config` declara `<staticContent>` com `mimeMap` para `.glb` (`model/gltf-binary`), `.gltf` (`model/gltf+json`), `.bin` (`application/octet-stream`) e `.ktx2` (`image/ktx2`). Sem esses MIME o IIS devolve **404.3** para qualquer arquivo em `/assets/models/*` — é a fase 4 do plano do mascote JOTA 3D; o build copia o arquivo de `public/` para `browser/` e o mirror publica na raiz do site.
- O URL Rewrite deve estar instalado.
- O mirror remove arquivos do site que não existem no pacote novo, preservando a árvore sincronizada.
- `three@0.186.1` é dependência de produção do `frontend/package.json` (`@types/three@0.186.0` em devDependencies), mas só entra no chunk lazy gerado por `jota-3d.loader.ts`: com o gate `JOTA_3D_HABILITADO = false` (estado atual) o bundle inicial não carrega `three`. **Pendências das fases 5 e 6** (plano em [`planos/jota-3d/00-PLANO-E-ESFORCO.md`](../../planos/jota-3d/00-PLANO-E-ESFORCO.md)): o GLB `frontend/src/assets/models/jota/jota.glb` ainda não existe e a ligação do gate em `jota-3d.types.ts` só deve ocorrer depois do modelo validado e do smoke servindo o `.glb`. Não há migration nem alteração de backend associadas.

