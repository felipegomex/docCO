---
id: sec.05-4-backend-net
titulo: "4. Backend .NET"
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
tokens_estimados: 1571
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 4. Backend .NET

### Desenvolvimento

```powershell
Set-Location .\backend\Central_BackEnd
dotnet restore
dotnet run
```

O perfil `http` inicia em `http://0.0.0.0:1009` com `ASPNETCORE_ENVIRONMENT=Development`. Acesse localamente por `http://localhost:1009`.

O `validate.ps1` executa `dotnet build --nologo` sem `-c Release` (3/7), `dotnet test` da suíte backend (5/7) e `ng test` (6/7), portanto valida o build Debug e os testes, mas não o build Release. O deploy executa `dotnet publish -c Release`; assim, o validate é um gate de compilação, mas não prova os binários Release exatos. Se o package falhar, pare: ele é o gate real de geração do artefato publicado.

### DiagnÃ³stico

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `SDK not found` ou projeto `net8.0` incompatÃ­vel | SDK 8 ausente | Instale o SDK 8 e confirme `dotnet --list-sdks` |
| backend encerra ao iniciar com erro de JWT | `JWT_KEY` e `Jwt:Key` indisponÃ­veis ao processo | ForneÃ§a a configuraÃ§Ã£o por fonte segura; nÃ£o registre o valor |
| backend encerra com tabela/columna ausente em Development + SQL | seeds de desenvolvimento consultam o schema antes de as tabelas existirem | Aplique as migrations no banco selecionado e reinicie |
| `address already in use` na 1009 | outro backend deste projeto estÃ¡ ativo | Identifique o processo pela linha de comando e encerre somente a instÃ¢ncia antiga |
| `/swagger` retorna 404 | `SwaggerEnabled` efetivo estÃ¡ falso | Verifique `appsettings*.json` e `ASPNETCORE_SWAGGER_ENABLED`; reinicie/recicle o processo |
| `/swagger/v1/swagger.json` retorna 500 "Erro interno no servidor" | Conflito de schemaId no Swashbuckle: DTOs com mesmo nome em namespaces diferentes (ex.: `Dtos.Shared.FuncaoResumo` vs `Dtos.Implantacao.FuncaoResumo`, `EtapaFuncaoResumo`, `OperadorResumo`) geram `InvalidOperationException: Can't use schemaId "$X" for type...` | Em `Program.cs`, no `AddSwaggerGen`, adicione `options.CustomSchemaIds(type => type.FullName?.Replace("+", "."))` para usar o nome completo do tipo (com namespace) como schemaId, garantindo unicidade |
| `/swagger/v1/swagger.json` retorna 500 e o log mostra `[FromForm] attribute used with IFormFile` | parâmetro `IFormFile` anotado com `[FromForm]` no controller | remova o `[FromForm]` do parâmetro `IFormFile` (o Swashbuckle trata o arquivo automaticamente); corrigido no commit `b6fa147` em `WhatsAppAtendimentosController.EnviarMidia` |
| rotas versionadas retornam 404 | versÃ£o/caminho incorreta | Use `/api/v1/...` ou envie `X-Api-Version: 1.0` |
| 500 ao fazer login | banco ausente, seed/consulta de login falhando ou configuraÃ§Ã£o de token inconsistente | Corrija o banco/configuraÃ§Ã£o e leia o stack no log; nÃ£o repita login em massa |
| Card de agenda da Home **sempre vazio**, e o log mostra `500` em `GET /api/v1/agenda/eventos` com `ArgumentException ... Kind property set correctly` em `TimeZoneHelper.BrasiliaParaUtc` | o frontend envia `inicio`/`fim` em ISO com `Z` (`Date.toISOString()`, `Kind=Utc`) e o helper antigo sÃ³ aceitava sem fuso; o erro era engolido pelo componente (`agenda.set([])` / `catchError(of([]))`) | corrigido em `Services/Implantacao/TimeZoneHelper.cs`: `Kind=Utc` passa intacto, sem fuso = BrasÃ­lia, `Kind=Local` usa `ToUniversalTime()`; regresÃ£o coberta por `TimeZoneHelperTests` (`dotnet test` em `backend/Central_BackEnd.Tests`). Se reaparecer, compare com esse teste antes de mexer no frontend |
| `GET /api/v1/agenda/eventos` com linhas, `POST /eventos` e `POST /eventos/lote` com 500 no servidor, mas `GET /tipos\|operadores\|funcoes`, `GET /eventos/{id}` inexistente (404) e `GET /eventos?responsavelId=ZZZ_INEXISTENTE` (200 `[]`) passam | `tbfuncionario.FUNCAO_ID` Ã© `smallint` no banco, mas `FuncionarioLegado.FuncaoId` era `int?` â†’ `InvalidCastException` Int16â†’Int32 em toda leitura que materializava a funÃ§Ã£o (listar com linhas, conflito, `FuncaoPropriaAsync`); lista vazia nÃ£o materializa e passa, e o INSERT do lote passava com a falha vindo depois (pÃ³s-leitura) | isolar lista-vazia vs com-linhas (`responsavelId` inexistente Ã— intervalo com dados); conferir o tipo real em `INFORMATION_SCHEMA.COLUMNS` (`tbfuncionario.FUNCAO_ID=smallint`, `tboperador.FUNCAO_ID=int`); InMemory nÃ£o checa tipos do provedor â€” validar contra SQL Server. Corrigido no commit `0d72b9e`: `FuncaoId short?` com casts `(int?)` em `AgendaService` (2 projeÃ§Ãµes, 2 comparaÃ§Ãµes) e `GestorFuncaoService`. O `12bb3d1` continua vÃ¡lido no que trata de sintoma (fuso fora da query, `LogError` com `detalhe = ex.Message` em Development, lote com `400` sem dias e `201` explÃ­cito sem id) |
| processo local funciona, IIS nÃ£o | ambiente, string de conexÃ£o, permissÃµes ou `web.config` diferentes | Compare apenas nomes de configuraÃ§Ã£o e estrutura; nÃ£o copie valores sensÃ­veis entre ambientes |

### Origem da chave JWT (fonte Ãºnica)

**Resolvido em `57a7e4f`** (regressÃ£o coberta por `f8a05ec`). Hoje a chave tem **uma Ãºnica fonte resolvida**: `JwtKeyProvider` (singleton) resolve `JWT_KEY ?? Jwt:Key` **uma vez** no startup. `Program.cs` (validaÃ§Ã£o) e `AuthService.GerarJwt` (emissÃ£o) consomem a mesma `SymmetricSecurityKey`, entÃ£o emissÃ£o e validaÃ§Ã£o nÃ£o divergem mais.

O que ainda exige cuidado manual:

- **Ambiente da mÃ¡quina, nÃ£o do pool.** `JWT_KEY` no ambiente do *application pool* Ã© apagado a cada recycle do IIS e derruba todas as sessÃµes. Mova-a para o ambiente da **mÃ¡quina**.
- **Nunca versionar.** A chave entra por variÃ¡vel de ambiente ou `user-secrets`; o arquivo versionado guarda sÃ³ placeholder (`ChaveApi: ""`).
- **DiagnÃ³stico no startup.** O log traz `JWT: chave resolvida de {Origem} (sha256:..., N bytes)` e avisa quando `JWT_KEY` e `Jwt:Key` divergem (`JWT_KEY` vence) ou quando a chave efetiva Ã© o placeholder â€” o Ãºltimo caso significa que qualquer instÃ¢ncia valida tokens alheios e exige correÃ§Ã£o imediata.

