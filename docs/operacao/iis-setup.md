---
title: "Setup IIS — Central de Operação"
description: "Instalação e configuração completa do IIS para rodar frontend Angular 18 + backend ASP.NET Core 8"
date: "2026-10-14"
language: "pt-BR"
status: "canônico"
---

# Setup IIS — Central de Operação

Guia consolidado para preparar um Windows Server (2019/2022) para receber o deploy do monorepo **Central de Conhecimento**.

> **Pré-requisito:** Role **Web Server (IIS)** já instalada via Server Manager.

---

## 1. Componentes Obrigatórios

| Componente | Versão | Verificação | Instalação |
|------------|--------|-------------|------------|
| **ASP.NET Core Hosting Bundle** | .NET 8 (LTS) | `dotnet --list-runtimes \| Select-String "8.0"` | [Download](https://dotnet.microsoft.com/download/dotnet/8.0) → "Hosting Bundle" |
| **URL Rewrite Module** | 2.1+ | `Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"` | [Download](https://www.iis.net/downloads/microsoft/url-rewrite) |
| **ANCM (ASP.NET Core Module)** | Vem no Hosting Bundle | `Test-Path "C:\Program Files\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll"` | — |

> **Importante:** Após instalar o Hosting Bundle, **reinicie o IIS** (`iisreset /restart`) para carregar o ANCM.

---

## 2. Script Automatizado

Localização: `scripts/iis-setup/setup-iis.ps1`

### Uso básico (baixa e instala tudo)

```powershell
# Como Administrador
Set-ExecutionPolicy Bypass -Scope Process -Force
.\scripts\iis-setup\setup-iis.ps1
```

### Parâmetros úteis

| Parâmetro | Padrão | Descrição |
|-----------|--------|-----------|
| `-DotNetHostingBundleUrl` | URL oficial .NET 8.0.10 | Altere se precisar de versão específica |
| `-UrlRewriteUrl` | URL oficial Microsoft | Altere se usar mirror interno |
| `-SiteBackPath` | `C:\inetpub\wwwroot\Suporte_Back` | Pasta do backend |
| `-SiteFrontPath` | `C:\inetpub\wwwroot\Suporte_Front` | Pasta do frontend |
| `-PortBack` | `1009` | Porta backend |
| `-PortFront` | `1010` | Porta frontend |
| `-AppPoolName` | `Suporte_Back` | Nome do App Pool |
| `-SkipDownload` | `$false` | Pula download (instalação offline) |
| `-Force` | `$false` | Recria App Pool/sites se existirem |

### Exemplo personalizado

```powershell
.\scripts\iis-setup\setup-iis.ps1 `
  -DotNetHostingBundleUrl "https://meu-interno/dotnet-hosting-8.0.11-win.exe" `
  -UrlRewriteUrl "https://meu-interno/rewrite_amd64_en-US.msi" `
  -SiteBackPath "D:\Sites\Suporte_Back" `
  -SiteFrontPath "D:\Sites\Suporte_Front" `
  -Force
```

---

## 3. O que o Script Faz

1. **Valida** se é Admin e se IIS base está instalado
2. **Instala URL Rewrite** (MSI silencioso)
3. **Instala Hosting Bundle .NET 8** (EXE silencioso)
4. **Verifica ANCM** carregado
5. **Cria App Pool** `Suporte_Back`:
   - No Managed Code (`managedRuntimeVersion = ""`)
   - 64-bit (`enable32BitAppOnWin64 = false`)
   - Identity: `ApplicationPoolIdentity`
6. **Cria pastas** físicas dos sites
7. **Cria sites IIS**:
   - `Suporte_Back` → porta 1009 → App Pool `Suporte_Back`
   - `Suporte_Front` → porta 1010 → `DefaultAppPool`
8. **Cria regras de firewall** entrada TCP 1009 e 1010
9. **Lista variáveis de ambiente** necessárias no App Pool (não define valores — segredos)

---

## 4. Variáveis de Ambiente (App Pool `Suporte_Back`)

Configure **após** o script, no IIS Manager → Application Pools → `Suporte_Back` → Advanced Settings → Environment Variables:

| Variável | Obrigatória? | Origem | Observação |
|----------|--------------|--------|------------|
| `JWT_KEY` | Sim | Secret manager / variável servidor | Base64 32 bytes |
| `Acessos__ChaveCifra` | Sim | Secret manager | Base64 32 bytes |
| `ConnectionStrings__DefaultConnection` | Sim* | Secret manager / appsettings.json | *Ou use `appsettings.json` na pasta do site |
| `DB_EXPLORER_SERVIDOR` | Não | Secret manager | Para Database Explorer |
| `DB_EXPLORER_PORTA` | Não | Secret manager | Padrão 1433 |
| `DB_EXPLORER_BANCO` | Não | Secret manager | |
| `DB_EXPLORER_USUARIO` | Não | Secret manager | |
| `DB_EXPLORER_SENHA` | Não | Secret manager | |
| `DB_EXPLORER_ENCRYPT` | Não | Secret manager | `true`/`false` |
| `DB_EXPLORER_TRUST` | Não | Secret manager | `true`/`false` |

> **Nunca** coloque segredos no script, em `appsettings.json` versionado ou em variáveis de ambiente do usuário. Use gerenciador de segredos aprovado.

---

## 5. Pós-Instalação (Deploy)

Após rodar o script, o pipeline padrão é:

```powershell
# 1. Na estação de trabalho (build + package)
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
powershell -ExecutionPolicy Bypass -Command "& '.\scripts\deploy\deploy.ps1' -BuildFrontend `$true -BuildBackend `$true -Publicar `$false"

# 2. Publicar no servidor (requer credencial)
$cred = Get-Credential -UserName "DOMAIN\usuario" -Message "Credencial IIS"
& '.\scripts\deploy\deploy.ps1' -BuildFrontend `$false -BuildBackend `$false -Publicar `$true -Backup `$true -UsuarioRemoto $cred.UserName -SenhaRemota $cred.Password

# 3. Smoke test
powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1 -Servidor 192.168.2.130
```

Ver detalhes em [`docs/10-DEPLOY.md`](../../docs/operacao/deploy.md).

---

## 6. Troubleshooting Rápido

| Sintoma | Causa | Solução |
|---------|-------|---------|
| `500.19` / `0x8007000d` | ANCM não instalado/carregado | Reinstale Hosting Bundle + `iisreset` |
| `500.30` / `500.31` | App Pool errado / .NET 8 ausente | Confirme App Pool = `Suporte_Back`, No Managed Code; `dotnet --list-runtimes` |
| `404` em rotas Angular (refresh) | URL Rewrite ausente | Instale URL Rewrite; confirme `web.config` na raiz do frontend |
| `401`/`403` no Swagger | `ASPNETCORE_SWAGGER_ENABLED` falso | `Set-ItemProperty "IIS:\AppPools\Suporte_Back" environmentVariables @{ASPNETCORE_SWAGGER_ENABLED="true"}` |
| Porta 1009/1010 não responde | Firewall | Script cria regras; verifique `Get-NetFirewallRule -DisplayName "CentralConhecimento_Porta_*"` |
| Variáveis não lidas | App Pool não reciclado | `Restart-WebAppPool -Name "Suporte_Back"` |

---

## 7. Arquivos de Configuração no Servidor

### Backend (`C:\inetpub\wwwroot\Suporte_Back\appsettings.json`)
```json
{
  "ConnectionStrings": { "DefaultConnection": "" },
  "Jwt": { "Key": "" },
  "Acessos": { "ChaveCifra": "" },
  "SwaggerEnabled": true,
  "Database": { "UseSqlServer": true }
}
```
> Valores vazios = placeholders. Preencha via variáveis de ambiente no App Pool (preferencial) ou edite este arquivo **após** o deploy (o script de deploy preserva `appsettings*.json`).

### Frontend (`C:\inetpub\wwwroot\Suporte_Front\web.config`)
Já vem no build (`frontend/public/web.config` → `dist/.../browser/web.config`). Contém regra de fallback SPA para URL Rewrite.

---

## 8. Checklist de Validação

- [ ] `dotnet --list-runtimes` mostra `Microsoft.AspNetCore.App 8.0.x`
- [ ] `Test-Path "C:\Windows\System32\inetsrv\rewrite.dll"` → `True`
- [ ] App Pool `Suporte_Back` existe, .NET CLR Version = "No Managed Code"
- [ ] Sites `Suporte_Back` (1009) e `Suporte_Front` (1010) rodando
- [ ] Variáveis de ambiente definidas no App Pool
- [ ] `appsettings.json` no backend do servidor preenchido (ou env vars equivalentes)
- [ ] Firewall permite entrada 1009 e 1010
- [ ] Deploy executado com sucesso (`deploy.ps1 -Publicar $true`)
- [ ] Smoke test passa: frontend 1010 + Swagger 1009

---

## 9. Referências

- [`docs/10-DEPLOY.md`](../../docs/operacao/deploy.md) — Pipeline completo de deploy
- [`docs/09-TROUBLESHOOTING.md`](../../docs/operacao/troubleshooting.md) — Diagnóstico detalhado
- [`scripts/deploy/deploy.ps1`](../../docs/../scripts/deploy/deploy.ps1) — Script de publicação
- [`scripts/smoke.ps1`](../../docs/../scripts/smoke.ps1) — Teste de fumaça