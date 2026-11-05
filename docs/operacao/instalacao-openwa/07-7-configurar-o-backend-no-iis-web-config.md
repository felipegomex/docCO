---
id: instalacao-openwa.07
titulo: "7. Configurar o backend no IIS (`web.config`)"
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
tokens_estimados: 951
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 7. Configurar o backend no IIS (`web.config`)

O backend recebe as variáveis do módulo por **variáveis de ambiente** injetadas pelo **ASP.NET Core Module (ANCM)**. No `web.config` do backend publicado (`C:\inetpub\wwwroot\Suporte_Back\web.config` na homologação), o bloco **precisa ser filho de `<aspNetCore>`** e usar o elemento `<environmentVariable name= value=>`:

```xml
<aspNetCore processPath="dotnet" arguments=".\Central_BackEnd.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="WhatsApp__Habilitado" value="true" />
    <environmentVariable name="WhatsApp__BaseUrl" value="http://192.168.2.253:2785/api" />
    <environmentVariable name="WhatsApp__WebhookUrl" value="http://192.168.2.130:1009/api/v1/whatsapp/webhook/openwa" />
    <environmentVariable name="WhatsApp__ApiKey" value="SUA_CHAVE" />
    <environmentVariable name="WhatsApp__WebhookSecret" value="SUA_SEGREDO" />
  </environmentVariables>
</aspNetCore>
```

O arquivo versionado [`backend/Central_BackEnd/wwwroot/web.config`](../../../backend/Central_BackEnd/wwwroot/web.config) usa **esse mesmo** schema (`<environmentVariables>` como filho de `<aspNetCore>`) — é dele que o `dotnet publish` deriva o `web.config` publicado. **As chaves e o segredo não entram no arquivo versionado**: só os placeholders.

### 7.1 O bloco é **regerado a cada publish** — reaplicar depois do deploy

O `deploy.ps1` espelha o backend com `/MIR` excluindo **apenas** `appsettings*.json` e `app_offline.htm`: o `web.config` **é sobrescrito** a cada publicação, e o bloco `<environmentVariables>` manual **segue junto**. Consequência operacional:

> **Com o módulo WhatsApp ativo, o bloco `<environmentVariables>` do `web.config` do backend precisa ser reaplicado DEPOIS de cada deploy.** Sem isso, o backend sobe sem `WhatsApp__BaseUrl`/`WhatsApp__ApiKey`/`WhatsApp__WebhookSecret` e o canal cai.

⚠️ **O mesmo deploy tem uma segunda metade: o schema.** Além do bloco do `web.config`, a publicação do módulo pode vir acompanhada de **mudança no script de banco** (por exemplo, o alargamento de `WAT_Status` para `varchar(50)`, §6.3). O `deploy.ps1` **não** aplica DDL: o script precisa ser **reaplicado** no banco do ambiente, **antes** do publish, sempre que houver mudança de largura ou de coluna. Sem isso, o código novo roda contra o schema antigo e o erro 8152 volta.

### 7.2 Reciclagem

Depois de editar o `web.config` na mão, o processo precisa carregar as variáveis novas:

1. **criar e remover `app_offline.htm`** na pasta do backend — é o mecanismo que o próprio `deploy.ps1` usa para tirar o site do ar e trazê-lo de volta; ou
2. **recyclar o application pool** do backend no IIS.

### 7.3 Diagnóstico do schema errado

Escrever o bloco no schema de `applicationHost.config` (`<environmentVariables><add key= value=/></environmentVariables>` como **irmão** de `<aspNetCore>`) **quebra o site inteiro**: o ANCM não sobe o processo e **todas** as rotas respondem `500` com **corpo vazio** — inclusive rotas que nem existem (`/naoexiste` deveria ser `404`).

O sintoma é **silencioso**: o evento do ANCM no Event Log (`1012`/`1032`/`1033`) é **normal** (start/recycle/shutdown) e o `stdout` do app **não** aparece, porque em hosting in-process o console não é anexado. O detector confiável é o `smoke.ps1`, que falha no check **"Swagger/API"**.

Diagnóstico completo em [`troubleshooting.md`](../troubleshooting.md) §13.1.

