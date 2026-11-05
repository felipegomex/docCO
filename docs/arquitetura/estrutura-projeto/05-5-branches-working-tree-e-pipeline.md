---
id: estrutura-projeto.05
titulo: "5. Branches, working tree e pipeline"
tipo: arquitetura
area: processo
sistema: central
tags: [monorepo, convencoes, pipeline, agents]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - AGENTS.md
  - scripts/validate.ps1
testes:
  - docs/
tokens_estimados: 743
ficha: docs/modulos/kanban.md
---
> Parte de [00 — Estrutura, leitura e fluxo de trabalho](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## 5. Branches, working tree e pipeline

### Branches

- `master` é a branch de release estável.
- `developer` é a branch de desenvolvimento integrada.
- Tags `vX.Y.Z` são releases imutáveis e substituem branches manuais de backup.
- Branches de backup encontradas localmente não participam do fluxo diário.

O working tree pode conter mudanças não commitadas e é a baseline considerada. `scripts/validate.ps1` mostra a situação Git e alerta sobre alterações, mas o alerta de working tree sujo não equivale a falha de build.

### Fluxo obrigatório documentado

```text
subir interno
  → validar
  → git add/commit/push em developer
  → validar novamente
  → empacotar
  → publicar com backup
  → smoke
```

Comandos de referência:

```powershell
# Gate de build; não sobe servidores nem publica no IIS
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1

# Empacotamento; ainda não publica no IIS
powershell -ExecutionPolicy Bypass -Command "& '.\scripts\deploy\deploy.ps1' -BuildFrontend 1 -BuildBackend 1 -Publicar 0"

# Publicação do pacote já empacotado, com backup
powershell -ExecutionPolicy Bypass -Command "& '.\scripts\deploy\deploy.ps1' -BuildFrontend 0 -BuildBackend 0 -Publicar 1 -Backup 1"

# Smoke
powershell -ExecutionPolicy Bypass -File .\scripts\smoke.ps1
```

Regras operacionais relevantes:

- Não iniciar publicação sem gate `validar` verde no commit atual.
- `scripts/deploy/deploy.ps1` grava `deploy/BUILD_INFO.txt` com hash e branch quando constrói o pacote.
- A publicação sem rebuild aborta se o hash de `BUILD_INFO.txt` divergir do `HEAD`.
- A publicação deve usar `-Backup 1` no fluxo canônico.
- O mirror do backend exclui `appsettings*.json`; a configuração existente no servidor é preservada.
- No script, `[Nullable[bool]]$BuildFrontend = $null` e `[Nullable[bool]]$BuildBackend = $null` fazem o build opcional perguntar quando não forem informados; `[bool]$Backup = $false` é o parâmetro de backup. Em chamadas PowerShell que usam valores `$true`/`$false`, use `-Command` e escape o `$` para o shell externo; os exemplos acima usam `1`/`0` para evitar a ambiguidade. A forma com backup explícito é:

```powershell
powershell -ExecutionPolicy Bypass -Command "& '.\scripts\deploy\deploy.ps1' -BuildFrontend `$false -BuildBackend `$false -Publicar `$true -Backup:`$true"
```

No comando acima, o escape impede que o shell externo interpole os valores; a opção do script é `-Backup:$true` (o valor não é um segredo).

- O script contém uma credencial padrão para autenticação no IIS; o valor não é reproduzido neste documento.
- Estes comandos estão documentados; sua execução não é afirmada neste conjunto.

