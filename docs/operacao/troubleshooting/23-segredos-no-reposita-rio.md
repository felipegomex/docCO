---
id: sec.23-segredos-no-reposita-rio
titulo: "Segredos no repositÃ³rio"
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
tokens_estimados: 1320
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# Segredos no repositÃ³rio

### Segredo em arquivo **publicado no servidor** (fora do Git)

âš ï¸ **Pendente em 2026-10-21:** o `appsettings.json` **publicado** do backend no servidor `192.168.2.254` contÃ©m `Jwt:Key` em **texto claro**. O `AGENTS.md` exige que a chave de JWT venha da **variÃ¡vel de ambiente `JWT_KEY` no servidor**. O valor atual deve ser considerado **comprometido** e a rotaÃ§Ã£o faz parte da F7 do plano de atendimento WhatsApp. O `appsettings.json` **do repositÃ³rio** estÃ¡ correto (sÃ³ o placeholder) â€” o problema Ã© a cÃ³pia publicada no servidor, que o `deploy.ps1` preserva. A mesma revisÃ£o confirmou que a chave da API do gateway (`WhatsApp__ApiKey`) e o segredo do webhook (`WhatsApp__WebhookSecret`) **nÃ£o** estÃ£o no repositÃ³rio nem no `appsettings.json` versionado.

> **DecisÃ£o do dono (2026-10-22):** a `Jwt:Key` em texto claro **permanece** no `appsettings.json` do servidor (`192.168.2.130`) por enquanto â€" nunca foi alterada nesta sessÃ£o; sem rotaÃ§Ã£o e sem `JWT_KEY` no ambiente da mÃ¡quina (o dono nÃ£o quer variÃ¡vel via HKLM agora). A migraÃ§Ã£o para `JWT_KEY` (ambiente da mÃ¡quina â€" o `JwtKeyProvider` jÃ¡ implementa a precedÃªncia) **continua pendente** e deve vir acompanhada da rotaÃ§Ã£o da chave.

### `validate.ps1` acusa segredo no passo 1/5

**Sintoma.** O gate para antes do build e imprime `arquivo:linha`, o padrÃ£o que casou e o **valor mascarado** (`se***23`). O script nÃ£o imprime o segredo inteiro â€” se ele aparecesse inteiro no console, o verificador teria criado um segundo vazamento.

**Como o verificador decide.** Ele varre **o Ã­ndice do Git** (`git diff --cached`), nÃ£o a Ã¡rvore de trabalho: o que importa Ã© o que seria commitado. Se acusou algo que vocÃª jÃ¡ corrigiu em disco, faltou `git add` â€” o Ã­ndice ainda tem a versÃ£o antiga.

**CorreÃ§Ã£o â€” nÃ£o troque por outro segredo hardcoded:**

1. Mova o valor para `user-secrets` ou variÃ¡vel de ambiente.
2. No arquivo versionado, deixe **sÃ³ o placeholder**: `""` no `appsettings.json`, `SUA_CHAVE` no `.sample.json`, `***` nos scripts.
3. Se o segredo **jÃ¡ esteve** no repositÃ³rio, remover o arquivo **nÃ£o basta** â€” rotacione (roteiro abaixo).

**Falso positivo conhecido.** `mkdir -p /opt/...` casava o padrÃ£o de senha em linha de comando; foi corrigido para ignorar valor que comeÃ§a com `/` ou `.`. Placeholders (`SUA_*`, `SEU_*`, `<no git>`, `***`, `${VAR}`) e identificadores sem dÃ­gito (`Senha`, `req.password`) sÃ£o ignorados de propÃ³sito. Se um caso legÃ­timo for barrado, ajuste a lista em `check-secrets.ps1` â€” nÃ£o desligue o gate.

### Credencial exposta: o que fazer

Descoberto em 2026-10-11: a senha do login `bussiness` estava em texto puro em dois arquivos versionados (`archive/_inspect-colunas.ps1` e `scripts/importar_acessos_xlsx.py`), e a string aparecia em 13 caminhos ao longo do histÃ³rico.

**Precedente do projeto.** `scripts/deploy/deploy.ps1` jÃ¡ tinha passado por isso â€” o comentÃ¡rio nas linhas 8-9 registra que a senha fixa foi removida "por exposiÃ§Ã£o no histÃ³rico do Git" e passou a exigir `-SenhaRemota` explÃ­cito. A regra adotada aqui Ã© a mesma: **remover do cÃ³digo, nÃ£o reescrever histÃ³rico**.

**Roteiro de rotaÃ§Ã£o** (o login `bussiness` ainda usa a senha antiga):

1. Trocar a senha do login no SQL Server.
2. Atualizar o segredo local do backend:
   ```powershell
   cd backend\Central_BackEnd
   dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=...;Password=<NOVA>;..."
   ```
   âš ï¸ O `secrets.json` do user-secrets tem **precedÃªncia** sobre `appsettings.Development.json`. Trocar sÃ³ o `appsettings.Development.json` **nÃ£o** troca o banco em desenvolvimento.
3. `subir interno` e confirmar que a API responde â€” Ã© este passo que valida a rotaÃ§Ã£o.
4. SÃ³ entÃ£o a senha antiga estÃ¡ inutilizada.

âš ï¸ **Ordem importa em repositÃ³rio com mais acesso.** Remover o arquivo antes de rotacionar cria uma janela em que o segredo sumiu do repositÃ³rio mas a senha antiga continua valendo. Com repositÃ³rio privado e time restrito o risco Ã© baixo, mas a ordem correta Ã© sempre **rotacionar â†’ validar â†’ limpar**.

**O que resolve e o que nÃ£o resolve:**

| AÃ§Ã£o | Resolve? |
|---|---|
| Rotacionar a senha | **Sim.** Ã‰ a Ãºnica que invalida a credencial |
| Remover o arquivo do repositÃ³rio | Ajuda, mas nÃ£o desfaz cÃ³pias jÃ¡ feitas |
| `git filter-repo` / reescrever histÃ³rico | NÃ£o invalida nada; reescreve SHA de `v0.8.0-rc1`, `v1.3.1`, `v1.3.2`, `v1.4.1` e forÃ§a reclone do time |

**Estado em 2026-10-11:** os 2 arquivos foram removidos e `check-secrets.ps1` passou a ser o passo 1 do gate. A senha antiga continua no histÃ³rico de `developer` e nas 4 tags acima; **decisÃ£o pendente do dono do projeto: rotacionar**. As tags foram mantidas de propÃ³sito â€” sÃ£o pontos de retorno do deploy, e o `AGENTS.md` as define como imutÃ¡veis.

