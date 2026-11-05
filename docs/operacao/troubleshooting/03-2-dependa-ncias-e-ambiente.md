---
id: sec.03-2-dependa-ncias-e-ambiente
titulo: "2. DependÃªncias e ambiente"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 822
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 2. DependÃªncias e ambiente

### Matriz de portas

| Uso | URL/porta | Origem da configuraÃ§Ã£o |
|---|---|---|
| Frontend em desenvolvimento | `http://localhost:4200` | `ng serve` |
| Backend em desenvolvimento | `http://localhost:1009` | `Properties/launchSettings.json` |
| Frontend publicado (HML) | `http://192.168.2.130:1010` | binding IIS |
| Frontend publicado (PROD) | `http://192.168.2.254:1010` | binding IIS PROD |
| Swagger | `http://localhost:1009/swagger` ou `http://<servidor-do-ambiente>:1009/swagger` | `SwaggerEnabled` efetivo |
| API v1 em desenvolvimento | `http://localhost:1009/api/v1` | `src/environments/environment.ts` |
| API v1 publicada (HML) | `http://192.168.2.130:1009/api/v1` | `src/environments/environment.prod.ts` |
| API v1 publicada (PROD) | `http://192.168.2.254:1009/api/v1` | `runtime-config.json` do servidor |
| **Gateway OpenWA (WhatsApp)** | `http://192.168.2.253:2785/api` â€” VM Ubuntu `jca-linux`; porta liberada **apenas** para `192.168.2.130` e `192.168.2.254` | `WhatsApp:BaseUrl` por variÃ¡vel de ambiente (`WhatsApp__BaseUrl`); o `appsettings.json` versionado ainda traz o loopback do desenho antigo |

### Sintomas, causas e soluÃ§Ãµes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `npm`, `ng` ou pacote local nÃ£o encontrado | `frontend/node_modules` ausente ou incompleto | Entre em `frontend` e rode `npm ci`; use o mesmo Node do projeto e o `package-lock.json` |
| `dotnet` nÃ£o reconhecido | SDK 8 ausente ou `PATH` desatualizado | Instale o SDK 8, reabra o terminal e rode `dotnet --info` |
| `dotnet ef` nÃ£o reconhecido | CLI global do EF Core ausente | Instale uma versÃ£o compatÃ­vel com EF Core 8; o pacote `Microsoft.EntityFrameworkCore.Tools` nÃ£o instala a CLI global |
| Restore do NuGet falha | rede, proxy, TLS, feed ou cache | Confirme acesso Ã  internet/rede interna, valide proxy/certificados e tente `dotnet restore` no projeto |
| `npm ci` falha com arquivo em uso | `ng serve`, build ou antivÃ­rus segurando `node_modules` | Feche apenas os processos deste frontend e repita |
| Backend nÃ£o alcanÃ§a o SQL Server do mÃ³dulo Acessos | rota, firewall ou `Database:UseSqlServer=false` (InMemory) | confirme a flag do ambiente e a conectividade; em InMemory os dados do mÃ³dulo nÃ£o sobrevivem ao processo |
| Deploy nÃ£o alcanÃ§a o servidor | rota, VPN, firewall ou DNS | Confirme conectividade atÃ© o servidor IIS do ambiente (`192.168.2.130` HML, `192.168.2.254` PROD; SQL Server Ã© `192.168.2.154`) e acesso administrativo ao compartilhamento `c$` |
| Script nem inicia | PowerShell incompatÃ­vel ou Execution Policy | Execute com `-ExecutionPolicy Bypass`; `validate.ps1` e `smoke.ps1` exigem PowerShell 5.1 ou superior |

PrÃ©-requisitos confirmados pelo projeto: Node.js 20 usado no workflow, npm, SDK .NET 8, Git, PowerShell, acesso aos feeds e, quando aplicÃ¡vel, `sqlcmd`/SSMS para migrations.

