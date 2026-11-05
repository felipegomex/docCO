---
title: "Setup local — Central de Operação"
description: "Onboarding do monorepo: pré-requisitos, clonagem, configuração de banco (UseSqlServer), subida do backend/frontend e troubleshooting de ambiente local."
date: "2026-10-19"
language: "pt-BR"
status: "canônico"
---

# Setup local — Central de Operação

Onboarding para rodar o monorepo **Central de Operação** (frontend Angular 18 + backend ASP.NET Core 8) na estação de trabalho. O setup do servidor IIS fica em [`docs/operacao/iis-setup.md`](./iis-setup.md).

Este documento é o **guia único de entrada** da estação de trabalho: ele absorveu o antigo `material-de-projeto/estagio/GUIA-SETUP.md` (guia de estagiários), consolidado aqui em 2026-10-20. O histórico do arquivo removido fica no Git.

- **Repositório:** <https://github.com/JCASOLUCOES/CENTRALOPERACAO_DEPLOY>
- **Issues:** <https://github.com/JCASOLUCOES/CENTRALOPERACAO_DEPLOY/issues>
- **Documentação (entrada única):** [`docs/README.md`](../README.md)
- **Deploy / troubleshooting:** [`docs/operacao/deploy.md`](./deploy.md) · [`docs/operacao/troubleshooting.md`](./troubleshooting.md)

## Pré-requisitos (instalar antes de clonar)

| Ferramenta | Versão mínima | Download | Verificação |
|------------|---------------|----------|-------------|
| Git | 2.40+ | <https://git-scm.com/download/win> | `git --version` |
| Node.js | 20.x LTS | <https://nodejs.org/pt-br/download/> | `node --version` |
| .NET SDK | 8.0.x | <https://dotnet.microsoft.com/download/dotnet/8.0> | `dotnet --version` |
| Angular CLI | 18.x | `npm install -g @angular/cli@18` | `ng version` |
| VS Code (recomendado) | Latest | <https://code.visualstudio.com/> | — |

> O SQL Server de homologação (`192.168.2.154` / `dbBUSINESS_HML`) exige acesso à rede interna (VPN). Sem VPN, use o banco InMemory (Passo 3).

## Passo a passo

### 1. Clonar o repositório

```bash
git clone https://github.com/JCASOLUCOES/CENTRALOPERACAO_DEPLOY.git
cd CENTRALOPERACAO_DEPLOY
```

### 2. Selecionar a branch de desenvolvimento

```bash
git checkout developer
```

Se aparecer `pathspec 'developer' did not match`:

```bash
git checkout -b developer origin/developer
```

### 3. Configurar o backend

`appsettings.Development.json` não é versionado (está no `.gitignore`). Parta do modelo `appsettings.sample.json`:

```bash
cd backend/Central_BackEnd
copy appsettings.sample.json appsettings.Development.json
```

O `appsettings.sample.json` **não tem** a seção `Database`. Acrescente no `appsettings.Development.json` local:

```json
"Database": { "UseSqlServer": false }
```

| Cenário | `Database:UseSqlServer` | Observação |
|---------|-------------------------|------------|
| Sem VPN | `false` (padrão) | InMemory, sem banco externo |
| Com VPN / rede interna | `true` | SQL Server via `ConnectionStrings:DefaultConnection`; exige VPN |

> O `dbBUSINESS_HML` em `192.168.2.154` é o banco de **produção** — não existe staging. Cada migração de schema é uma alteração em produção já na primeira vez.
>
> Fora de Development, o backend sempre usa `ConnectionStrings:DefaultConnection`.

### 4. Instalar dependências

```bash
# Backend
cd backend/Central_BackEnd
dotnet restore

# Frontend
cd ../../frontend
npm install
```

### 5. Configurar segredos locais (user-secrets)

O backend precisa de `Jwt:Key` (assinar/validar tokens) e de uma conexão em `ConnectionStrings:DefaultConnection`. Defina via `user-secrets` (nunca em arquivo versionado):

```bash
cd backend/Central_BackEnd
dotnet user-secrets set "Jwt:Key" "<chave local, 32 bytes ou mais>"
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "<conexão do homolog>"
```

> Os valores reais **não** são escritos neste guia; peça ao responsável pelo banco. Sem `Jwt:Key`, o login falha.

### 6. Subir o projeto (dois terminais)

Terminal 1 — backend:

```bash
cd backend/Central_BackEnd
dotnet run
```

Backend em <http://localhost:1009> (Swagger em <http://localhost:1009/swagger>).

Terminal 2 — frontend:

```bash
cd frontend
ng serve
```

Frontend em <http://localhost:4200>.

Alternativas:

| Opção | Comando | Quando usar |
|-------|---------|-------------|
| Skill `subir interno` | no opencode: `subir interno` | Preserva o interruptor de banco (InMemory × homolog) e sobe os dois processos |
| `make serve` | `make serve` (raiz) | Sobe backend e frontend em paralelo, com `dotnet watch` — recarrega ao salvar |
| VS Code | task `Central: Serve (All)` | Debug com breakpoint (`.vscode/launch.json`) |
| Só um lado | `make backend` · `make frontend` | Quando só um dos lados precisa ficar de pé |

> O antigo `scripts/deploy/run-dev.bat` foi **removido** (guardava a chave JWT em texto e caminho absoluto). Não volte a criá-lo: a skill `subir interno` cobre o mesmo uso.

## Verificar se subiu corretamente

| Serviço | URL | Esperado |
|---------|-----|----------|
| Backend API | <http://localhost:1009/swagger> | Swagger UI com a lista de endpoints |
| Frontend | <http://localhost:4200> | Tela de login da Central de Operação |

Login: usuário `admin` do seed local. A senha está definida no código de desenvolvimento (seed do administrador local em `backend/Central_BackEnd/Program.cs`); não é documentada aqui.

## Comandos úteis

```bash
# Gate de commit antes de commit (segredos + build + testes; obrigatório no fluxo)
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1

# Testes do backend (xUnit) — também rodam no passo 5/7 do validate
cd backend/Central_BackEnd.Tests
dotnet test

# Status do git
git status --short

# Atualizar a branch developer
git pull origin developer

# Diff das mudanças não commitadas
git diff

# Adicionar somente arquivos intencionais (não use `git add .`)
git add -- <arquivo1> <arquivo2>

# Commit (convenção)
git commit -m "fix: descrição curta da correção"

# Push
git push origin developer
```

### Convenção de commits

| Prefixo | Uso |
|---------|-----|
| `feat:` | Nova funcionalidade |
| `fix:` | Correção de bug |
| `docs:` | Apenas documentação |
| `chore:` | Manutenção (dependências, config, scripts) |
| `refactor:` | Refatoração sem mudança de comportamento |

## Problemas comuns

| Problema | Causa provável | Solução |
|----------|----------------|---------|
| `dotnet: command not found` | .NET SDK ausente ou PATH desatualizado | Reinicie o terminal; confira `dotnet --version` |
| `ng: command not found` | Angular CLI não instalado globalmente | `npm install -g @angular/cli@18` e reinicie o terminal |
| Erro de conexão SQL Server | VPN desconectada ou `UseSqlServer=true` sem acesso | Conecte a VPN ou volte `Database:UseSqlServer` para `false` |
| Porta 1009/4200 ocupada | Outro processo nas portas | `netstat -ano \| findstr :1009` e `taskkill /PID <PID> /F` |
| `The SDK 'Microsoft.NET.Sdk.Web' ... could not be found` | Workloads do .NET ausentes | `dotnet workload restore` ou reinstale o .NET 8 SDK |
| Frontend em branco / não carrega | Erro de compilação ou cache | Abra o console (F12); tente `ng serve --no-cache` |
| `Cannot find module '@angular/core'` | `node_modules` corrompido | `cd frontend && rm -rf node_modules package-lock.json && npm install` |
| Backend não compila (CS0246, CS0103) | Pacotes NuGet não restaurados | `cd backend/Central_BackEnd && dotnet restore` |

## Estrutura rápida do projeto

```text
CENTRALOPERACAO_DEPLOY/
├── backend/
│   ├── Central_BackEnd/            # ASP.NET Core 8 Web API
│   │   ├── Controllers/            # Endpoints REST
│   │   ├── Services/               # Regras de negócio
│   │   ├── Models/                 # Entidades EF Core + DTOs
│   │   └── Migrations/             # Migrations EF Core + scripts SQL
│   └── Central_BackEnd.Tests/      # Testes xUnit
├── frontend/
│   ├── src/app/
│   │   ├── features/               # Módulos lazy (uma pasta por feature)
│   │   ├── core/                   # Services, guards, interceptors
│   │   ├── shared/                 # Componentes/diretivas/pipes compartilhados
│   │   └── layouts/                # MainLayout, AuthLayout
│   └── angular.json                # Angular 18 (builder application, browser)
├── docs/                           # Documentação canônica
│   ├── README.md                   # Entrada única
│   ├── arquitetura/                # Visão geral, componentes, banco de dados, estrutura
│   ├── referencia/                 # Regras, endpoints, frontend, backend, QA
│   └── operacao/                   # Deploy, troubleshooting, setup local, IIS
├── scripts/
│   ├── validate.ps1                # segredos + dotnet build + ng build + dotnet test (gate obrigatório)
│   ├── smoke.ps1                   # Health check pós-deploy
│   ├── deploy/deploy.ps1           # Empacotar e publicar no IIS
│   └── iis-setup/setup-iis.ps1     # Instalação do IIS no servidor
├── material-de-projeto/             # Fora do canônico: ideias (backlog do módulo), backup (diagnóstico e rollback)
└── AGENTS.md                       # Regras para agentes IA (fluxo obrigatório)
```

## Fluxo de trabalho (resumo)

Ao concluir uma tarefa, siga esta ordem:

```text
1. subir interno   -> testa no navegador (dev, sem build; InMemory ou homolog)
2. validar         -> scripts/validate.ps1 (segredos + dotnet build + ng build + dotnet test) - se vermelho, pare
3. git add/commit/push -> grava no Git (branch developer)
4. validar         -> confere o commit
5. deploy limpo    -> empacota (BUILD_INFO) -> publica no IIS -> smoke
```

> Regra de ouro: nunca faça deploy limpo sem `validar` verde no commit atual.

| Etapa | Comando |
|-------|---------|
| Dev local | `dotnet run` (backend) + `ng serve` (frontend) |
| Gate de commit | `powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1` (7 passos) |
| Empacotar | `scripts/deploy/deploy.ps1 -BuildFrontend 1 -BuildBackend 1 -Publicar 0` |
| Publicar | `scripts/deploy/deploy.ps1 -BuildFrontend 0 -BuildBackend 0 -Publicar 1 -Backup 1` |
| Smoke test | `powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1` |

## Suporte e referências

| Dúvida | Onde consultar |
|--------|----------------|
| Setup local / erros de build | [`docs/operacao/troubleshooting.md`](./troubleshooting.md) |
| Setup do IIS no servidor | [`docs/operacao/iis-setup.md`](./iis-setup.md) |
| Pipeline de deploy / rollback / smoke | [`docs/operacao/deploy.md`](./deploy.md) |
| Fluxo de branches / versionamento | [`AGENTS.md`](../../AGENTS.md) |
| Estrutura do repositório | [`docs/arquitetura/estrutura-projeto.md`](../arquitetura/estrutura-projeto/INDEX.md) |
| Visão geral / regras de negócio | [`docs/arquitetura/visao-geral.md`](../arquitetura/visao-geral/INDEX.md) · [`docs/referencia/regras-negocio.md`](../referencia/regras-negocio/INDEX.md) |
| Estrutura de dados / entidades | [`docs/arquitetura/banco-dados.md`](../arquitetura/banco-dados.md) |
| Endpoints / API | [`docs/referencia/endpoints.md`](../referencia/endpoints.md) |
| Componentes / rotas do frontend | [`docs/referencia/frontend.md`](../referencia/frontend.md) |
| Services / controllers do backend | [`docs/referencia/backend.md`](../referencia/backend.md) |
| Cenários de QA / histórias | [`docs/referencia/qa-historias.md`](../referencia/qa-historias.md) |
| Issues | <https://github.com/JCASOLUCOES/CENTRALOPERACAO_DEPLOY/issues> |

## Checklist do primeiro dia

- [ ] Git, Node.js 20, .NET 8 e Angular CLI 18 instalados
- [ ] VPN configurada e testada (`ping 192.168.2.154`) — ou InMemory definido
- [ ] Repositório clonado e branch `developer` ativa
- [ ] `appsettings.Development.json` criado, com `Database:UseSqlServer` e `user-secrets` configurados
- [ ] `dotnet restore` e `npm install` sem erros
- [ ] Backend (1009) e frontend (4200) no ar
- [ ] Login em <http://localhost:4200> funcionando
- [ ] `scripts/validate.ps1` verde
- [ ] Leitura de [`AGENTS.md`](../../AGENTS.md) (seção Pipeline Dev/Deploy)
