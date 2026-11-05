---
id: sec.14-refresh-tokens-e-auditoria
titulo: "Refresh tokens e auditoria"
tipo: referencia
area: dados
sistema: central
tags: [schema, banco, tabelas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 778
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Refresh tokens e auditoria

### `tbrefreshtoken` (`RefreshTokens`)

- A tabela guarda `TokenHash`, não o token bruto. O operador é FK e a remoção do operador remove os tokens.
- `ExpiraEm`, `CriadoEm`, `Revogado` e `SubstituidoPor` suportam expiração, rotação e auditoria de uso.
- O hash usa SHA-256 do token. O token em claro só existe no cookie temporário e no retorno interno do serviço; o controlador não o remove do JSON: o campo é serializado como `refreshToken: ""` no login e no refresh.
- Não há endpoint de consulta de tokens; as operações de login, refresh e logout são as operações expostas.

### `tbauditoriaimplantacao` (`IMPL_Auditoria`)

- `AuditoriaImplantacaoService` serializa estado anterior/posterior em JSON, com `camelCase`, remoção de nulos e ignora ciclos.
- Falha ao registrar auditoria é apenas logada e não desfaz a operação principal.
- A cobertura é por serviço: criação/alteração/exclusão de projeto e tarefa, movimentação de tarefa, vínculo de chamado e criação de apontamento; não é um registro universal de todas as alterações de etapa/agenda.
- `HistoricoAsync` da tarefa lê essa auditoria e monta `HistoricoMovimentacao`. A leitura geral das duas trilhas é a tela `/auditoria`, servida por `GET /api/v1/auditoria/acessos` e `GET /api/v1/auditoria/implantacao` (`AuditoriaController` + `AuditoriaLeituraService`, ambos paginados via `Paginado<T>`) — ver o catálogo em [`05-ENDPOINTS.md`](../../referencia/endpoints.md).

### `tbauditoriaacesso` (`AuditoriaAcessos`)

- É a auditoria de **revelação de credencial** do módulo Acessos Empresas. Registra **quem**, **de qual empresa**, **qual tipo de acesso**, **qual registro**, **quando**, **de qual IP** e **com qual navegador**. **Nunca grava a senha** — nem cifrada nem em claro.
- As colunas `AAC_ClienteAcessoId`, `AAC_AcessoTipo` e `AAC_AcessoId` chegaram na migration `20260928212856_AcessosEmpresas`, **nuláveis**: as 58 linhas antigas (log pela empresa, sem grupo) continuam válidas e não são interpretadas pelo módulo novo. `AAC_ClienteAcessoId` tem índice porque é o filtro natural do relatório de auditoria.
- A coluna legada `Empresa` é `varchar(200) NOT NULL`; o processador escreve nela o nome da entidade para manter o texto legível, mas **o vínculo real é `AAC_*`**. Não faça consulta por `Empresa` para reconstruir o que foi revelado.
- A gravação é **assíncrona, por fila**. A tela não espera: a revelação responde e o registro é persistido em background. Se o processo morrer com a fila cheia, o evento se perde — o contador aparece em `GET /api/v1/acessos/estado-cifra` (`eventosAuditoriaPerdidos`) e o log traz `Error`. A leitura é a aba "acessos" da tela `/auditoria` (`GET /api/v1/auditoria/acessos`, paginado).

