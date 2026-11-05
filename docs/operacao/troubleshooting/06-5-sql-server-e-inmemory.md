---
id: sec.06-5-sql-server-e-inmemory
titulo: "5. SQL Server e InMemory"
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
tokens_estimados: 1356
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 5. SQL Server e InMemory

`Program.cs` escolhe o provider apenas no ambiente `Development`:

| `Database:UseSqlServer` | Provider | PersistÃªncia |
|---:|---|---|
| `false` | EF Core InMemory `CentralDev` | perde tudo ao encerrar o processo |
| `true` | SQL Server por `ConnectionStrings:DefaultConnection` | persiste no banco configurado |

Fora de `Development`, o backend usa SQL Server. Na baseline atual, `appsettings.Development.json` estÃ¡ configurado para `true`, portanto o dev usa SQL Homolog atÃ© a flag ser alterada.

### InMemory

Para forÃ§ar memÃ³ria efÃªmera:

1. Altere `Database:UseSqlServer` para `false` em `appsettings.Development.json`.
2. Reinicie o backend.
3. Confirme no log/terminal que a aplicaÃ§Ã£o iniciou com o provider em memÃ³ria.

O seed de desenvolvimento cria dados estruturais e operadores de teste. Credenciais de teste nÃ£o sÃ£o reproduzidas neste guia; obtenha-as por canal interno aprovado.

LimitaÃ§Ã£o: o provider InMemory nÃ£o valida SQL Server, FKs, Ã­ndices, colunas geradas, `OPENJSON` ou SQL especÃ­fico. Uma tela funcionar em memÃ³ria nÃ£o prova que o banco real estÃ¡ correto â€” no banco legado em compat 100 isso significa que o `OPENJSON` gerado pelo EF sÃ³ estoura em produÃ§Ã£o. Veja a seÃ§Ã£o "Dashboard Integrado contra o banco legado (compat 100)" deste guia.

### SQL Homolog

Para usar SQL Homolog:

1. Mantenha `Database:UseSqlServer` como `true`.
2. Garanta rota atÃ© `192.168.2.154` e acesso ao banco `dbBUSINESS_HML`.
3. ForneÃ§a a string de conexÃ£o por `user-secrets`/configuraÃ§Ã£o local protegida; nÃ£o a imprima.
4. Garanta que as migrations do commit atual jÃ¡ foram aplicadas.
5. Reinicie o backend apÃ³s mudar provider ou schema.

### Sintomas

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| dados desaparecem ao reiniciar | InMemory selecionado | troque para SQL somente quando quiser testar persistÃªncia real |
| timeout/login no SQL Server | rede, porta, autenticaÃ§Ã£o, TLS/certificado ou banco indisponÃ­vel | Teste rota e autenticaÃ§Ã£o com ferramenta segura; confira mensagem do SqlClient |
| â€œInvalid object nameâ€ | migration nÃ£o aplicada no banco selecionado | consulte `__EFMigrationsHistory`, aplique o script revisado e valide o schema |
| **â€œSintaxe incorreta prÃ³xima a 'GO'â€** ao aplicar um `.sql` por `System.Data.SqlClient` | o `GO` Ã© um **separador de lote do SSMS/sqlcmd**, nÃ£o Ã© SQL: o `SqlClient` envia o arquivo como **um** comando e o servidor rejeita a linha `GO` | quebre o arquivo em lotes nos separadores `GO` e execute cada lote separadamente (ver Â§5.1) |
| â€œInvalid column nameâ€ em tela do Dashboard Integrado | coluna inexistente no banco legado mapeada pelo espelho (ex.: `DATA_FATURAMENTO`, que na verdade Ã© `DIA_FATURAMENTO`) ou migration de coluna nÃ£o aplicada | consulte `INFORMATION_SCHEMA.COLUMNS`; **um** erro de coluna derruba todos os endpoints que leem a tabela â€” veja a seÃ§Ã£o 12 |
| tela Development funciona, produÃ§Ã£o falha | schemas e migrations divergentes | compare histÃ³rico e estrutura do banco alvo; nÃ£o copie dados entre bancos para â€œcorrigirâ€ |
| Database Explorer falha mesmo em InMemory | integraÃ§Ãµes usam SQL/rede prÃ³prios, nÃ£o o `AppDbContext` em memÃ³ria | diagnostique cada conexÃ£o separadamente |
| Homolog foi alterado sem querer | provider SQL selecionado | pare o backend, confirme a flag e revise dados/timestamps; use o script de limpeza apenas com backup e aprovaÃ§Ã£o |

### 5.1 Aplicar um script `.sql` em lotes (o `GO` nÃ£o Ã© SQL)

**Sintoma.** `System.Data.SqlClient` devolve **â€œSintaxe incorreta prÃ³xima a 'GO'â€** ao executar um arquivo `.sql` de migration.

**Causa.** `GO` Ã© um **separador de lote** do SSMS e do `sqlcmd` â€” nÃ£o faz parte da linguagem SQL. O `SqlClient` nÃ£o o interpreta: ele envia o arquivo inteiro como **um Ãºnico comando**, e o servidor rejeita a linha `GO` como sintaxe invÃ¡lida.

**SoluÃ§Ã£o.** Quebre o arquivo nos separadores `GO` e execute cada lote como um comando prÃ³prio (o caso real foi o [`Sql/WhatsApp_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Idempotente.sql), aplicado em `dbBUSINESS_HML` em 2026-10-21):

```csharp
var sql = File.ReadAllText(@"caminho\WhatsApp_Idempotente.sql");
var lotes = sql.Split(new[] { "\r\nGO\r\n", "\nGO\n", "\rGO\r" },
                       StringSplitOptions.None);

foreach (var lote in lotes)
{
    if (string.IsNullOrWhiteSpace(lote)) continue;
    using var cmd = new SqlCommand(lote, conexao);
    cmd.CommandTimeout = 120;
    cmd.ExecuteNonQuery();
}
```

Alternativas vÃ¡lidas: rodar o arquivo pelo `sqlcmd` (que entende o `GO`) ou pelo SSMS. O que **nÃ£o** funciona Ã© enviar o arquivo inteiro em uma chamada sÃ³.

âš ï¸ A aplicaÃ§Ã£o real foi feita **de dentro da rede**, lendo a connection string do `appsettings.json` do prÃ³prio servidor â€” nunca com a credencial na linha de comando. Sem backup prÃ©vio, por decisÃ£o explÃ­cita do dono do projeto (o script Ã© **aditivo** e idempotente: a reaplicaÃ§Ã£o Ã© no-op).

