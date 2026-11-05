---
id: sec.10-9-database-explorer
titulo: "9. Database Explorer"
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
tokens_estimados: 1000
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 9. Database Explorer

### ConfiguraÃ§Ã£o

O serviÃ§o lÃª, por prioridade de variÃ¡vel de ambiente, os nomes:

- `DB_EXPLORER_SERVIDOR`
- `DB_EXPLORER_PORTA`
- `DB_EXPLORER_BANCO`
- `DB_EXPLORER_USUARIO`
- `DB_EXPLORER_SENHA`
- `DB_EXPLORER_ENCRYPT`
- `DB_EXPLORER_TRUST`

A tela de configuraÃ§Ã£o Ã© somente leitura. O backend considera servidor, banco e usuÃ¡rio para dizer que estÃ¡ configurado, mas uma senha ausente sÃ³ aparece como falha ao abrir a conexÃ£o. O timeout de conexÃ£o Ã© 10 segundos.

Todos os endpoints `/api/v1/database/*` exigem Bearer. A senha Ã© mascarada em `GET /api/v1/database/config`; `GET /api/v1/database/status` executa `SELECT 1` e pode retornar HTTP 200 com `conectado: false`.

### Sintomas, causas e soluÃ§Ãµes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| 401 em qualquer tela do Database Explorer | sessÃ£o ausente/expirada | faÃ§a login e valide o fluxo do cookie; nÃ£o Ã© falha de SQL |
| 200 com `conectado: false` | timeout, autenticaÃ§Ã£o, TLS, rota ou banco | use a mensagem retornada e teste do host do backend, nÃ£o do navegador |
| â€œConexÃ£o SQL Server nÃ£o configuradaâ€ | servidor, banco ou usuÃ¡rio ausentes | configure os trÃªs no processo/IIS; a tela nÃ£o grava configuraÃ§Ã£o |
| senha mascarada, mas conexÃ£o recusada | `DB_EXPLORER_SENHA` ausente/incorreta no pool | forneÃ§a a variÃ¡vel por canal seguro e recicle `Suporte_Back` |
| timeout de 10 segundos | SQL Server indisponÃ­vel, porta bloqueada ou DNS | valide rota/firewall a partir do servidor IIS |
| certificado TLS recusado | `Encrypt`/`TrustServerCertificate` incompatÃ­veis | alinhe essas opÃ§Ãµes Ã  polÃ­tica do SQL Server sem desabilitar seguranÃ§a por soluÃ§Ã£o permanente |
| dados antigos apÃ³s alteraÃ§Ã£o de schema | backend ainda ativo com estado anterior | nÃ£o hÃ¡ cache de schema; reinicie/recicle e valide a conexÃ£o real |
| espera-se executar SQL digitado | nÃ£o existe endpoint de execuÃ§Ã£o livre | o mÃ³dulo atual consulta metadados e o Criador apenas gera/copia SQL |
| â€œN linha(s) do arquivo nao puderam ser lidas como tabela: Linha X: â€¦â€ (400 no banco inteiro) | arquivo JSON Lines/CSV-Lines com **linha genuinamente quebrada** (JSON truncado, aspas partidas etc.); o parse nunca perde linha em silÃªncio | a mensagem aponta o nÃºmero da linha e um trecho; corrija essa linha no arquivo e reenvie. **Caractere de controle nÃ£o Ã© mais motivo de 400**: `0x00-0x1F` em string JSON Ã© reparseado com escape `\u00XX` (nome preservado) |
| arquivo gerado pelo script com caractere de controle (ex.: `0x1F` no nome de tabela) | o script antigo (FOR XML PATH, SQL 2008) escapava sÃ³ `"` e `\`; controles saÃ­am crus e invalidavam o JSON | **nÃ£o edite o arquivo Ã  mÃ£o**: o parser reparseia com `\u00XX` e o nome com o caractere aparece como divergÃªncia visÃ­vel; o script de exportaÃ§Ã£o do sistema (versÃ£o nova) jÃ¡ emite JSON vÃ¡lido escapando os controles |
| â€œJSON invalido para banco inteiroâ€ (400) | raiz nÃ£o Ã© `{tabelas:[...]}`, array de objetos de tabela nem JSON Lines | exporte com o script de banco inteiro ou monte o formato esperado |
| tabela sumindo do comparador apÃ³s upload (aparecia antes de 2026-10) | parser antigo perdia em silÃªncio a 1Âª e a Ãºltima linha de CSV-Lines (unwrap Excel/SSMS corrompia extremidades) | corrigido no commit `bf20b65`; atualize o backend e o erro agora aponta a linha problemÃ¡tica |

O helper `scripts/db/configure-db-explorer-password.ps1` foi **removido** em 2026-10-14 (podia imprimir o segredo configurado e nÃ£o era canal seguro). Prefira configuraÃ§Ã£o protegida no Application Pool.

