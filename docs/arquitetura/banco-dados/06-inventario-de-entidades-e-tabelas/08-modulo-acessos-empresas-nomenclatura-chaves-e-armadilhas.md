---
id: 06-inventario-de-entidades-e-tabelas.08
titulo: "Módulo Acessos Empresas — nomenclatura, chaves e armadilhas"
tipo: referencia
area: dados
sistema: central
tags: [schema, tabelas, entidades, sqlserver]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
  - backend/Central_BackEnd/Migrations
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1702
ficha: docs/modulos/acessos.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### Módulo Acessos Empresas — nomenclatura, chaves e armadilhas

Prefixo de tabela `tb` + entidade em minúsculas, colado, **sem prefixo de módulo** — segue a regra geral do projeto. As colunas usam prefixo curto por tabela + `_` e PascalCase, como `tblead`/`LEAD_`:

| Classe | Tabela | Prefixo | Arquivo do modelo |
|---|---|---|---|
| `ClienteAcesso` | `tbclienteacesso` | `CAO_` | [ClienteAcesso.cs](../../../../backend/Central_BackEnd/Models/Acessos/ClienteAcesso.cs) |
| `AcessoRdp` | `tbacessordp` | `RDP_` | [Acessos.cs](../../../../backend/Central_BackEnd/Models/Acessos/Acessos.cs) |
| `AcessoVpn` | `tbacessovpn` | `VPN_` | [Acessos.cs](../../../../backend/Central_BackEnd/Models/Acessos/Acessos.cs) |
| `AcessoAnyDesk` | `tbacessoanydesk` | `ADK_` | [Acessos.cs](../../../../backend/Central_BackEnd/Models/Acessos/Acessos.cs) |
| `AcessoBanco` | `tbacessobanco` | `BCO_` | [Acessos.cs](../../../../backend/Central_BackEnd/Models/Acessos/Acessos.cs) |
| `AcessoActyonWeb` | `tbacessoactyonweb` | `AWB_` | [Acessos.cs](../../../../backend/Central_BackEnd/Models/Acessos/Acessos.cs) |

`AcessoBase` ([AcessoBase.cs](../../../../backend/Central_BackEnd/Models/Acessos/AcessoBase.cs)) é a classe abstrata comum aos 5 tipos. Ela **não** carrega atributos `[Column]`: as colunas reais ficam nas derivadas, cada uma com seu prefixo. O que a base carrega é apenas o nome/tipo idêntico (`Id`, `ClienteAcessoId`, `Cliente`, `Usuario`, `Observacao`, `Ativo`, `DataAlteracao`, `UsuarioAlteracao`) — e as derivadas redeclaram todos eles com o `[Column]` correto, exatamente pelo mesmo motivo dos atributos.

**Exceção que produz uma coluna sem prefixo:** `Usuario` é a única propriedade que a base declara com atributo (`[MaxLength(100)]`) mas **sem** `[Column]`. As 4 derivadas com senha (`AcessoRdp`, `AcessoVpn`, `AcessoAnyDesk`, `AcessoBanco`) **redeclaram** `Usuario` com o `[Column]` prefixado da sua tabela, o que sombreia a da base. `AcessoActyonWeb` **não** redeclara: herda a da base, e o EF mapeia por convenção para a coluna `Usuario`, sem prefixo. É assim que `tbacessoactyonweb.Usuario` existe e é legítima — **não remova essa coluna.** Ela não faz parte do contrato do Actyon Web (os DTOs de criação/alteração/detalhe não a expõem), mas é coluna mapeada e persistida pelo EF.

**Chaves e exclusões:**

- Todas as 7 FKs do módulo usam `ON DELETE NO ACTION` (= `Restrict`) no banco: `CAO_GrupoId` → `tbcliente_grupo`, `CAO_ClienteId` → `tbcliente` e as 5 de acesso → `tbclienteacesso`. O SQL Server recusa múltiplos caminhos em cascata, e o projeto já usa `Restrict` pelo mesmo motivo em `tbprojeto`/`tblead`. Quem desativa é a coluna `Ativo` (soft delete), nunca o banco. A FK antiga `CAO_GrupoId` → `tbgrupoacesso` foi removida junto com a tabela.
- ~~`CAO_Cnpj`~~ e ~~`CAO_Fantasia`~~ **não existem mais**: as colunas locais de identidade (e seus índices `IX_tbclienteacesso_CAO_Cnpj` único filtrado e `IX_tbclienteacesso_CAO_Fantasia`) foram dropadas pela `20261002161552_RemoveIdentidadeLocalAcessos`. A unicidade passou a ser **só** por `CLIENTE_ID`, e a identidade (fantasia, CNPJ, grupo) é lida da `tbcliente` por JOIN — o serviço normaliza o CNPJ do ERP para 14 dígitos só na exibição.
- `CAO_ClienteId` tem **índice único filtrado** (`WHERE CAO_ClienteId IS NOT NULL`): cada empresa do ERP (`tbcliente.CLIENTE_ID`) sustenta **no máximo uma** unidade; repetir a seleção da mesma empresa devolve a unidade existente (`JaExistia = true`) em vez de duplicar.
- As 5 flags `CAO_Tem*` são **derivadas**, nunca editáveis pelo consumidor: `AcessosService.RecalcularFlagsAsync` as recalcula a partir da existência de acesso ativo depois de cada mutação. São um índice desnormalizado para a lista paginada; se virarem campo editável, inativar o único RDP deixaria `CAO_TemRdp = true`.
- Índices do módulo: `IX_tbclienteacesso_CAO_GrupoId`, o único filtrado de `CAO_ClienteId`, um composto `(cliente, ativo)` por tabela de acesso e `IX_tbauditoriaacesso_AAC_ClienteAcessoId`. Saiu `IX_tbgrupoacesso_GAO_Nome` (drop da tabela) e, em `20261002161552`, saíram `IX_tbclienteacesso_CAO_Cnpj` e `IX_tbclienteacesso_CAO_Fantasia` — **8 índices**.

⚠️ **`SenhaCifrada` e `TemSenha` NÃO estão em `AcessoBase` — de propósito.** Se a base declarasse `SenhaCifrada`, a propriedade da derivada a **sombrearia** e o `TemSenha` derivado dela leria a propriedade da **base**, que nunca é preenchida, devolvendo sempre `false` mesmo com a senha gravada. Por isso `TemSenha` também é declarado em cada classe concreta. É a armadilha mais provável de "simplificação" neste módulo: **não mova essas duas propriedades para a base.** Regressão coberta por `backend/Central_BackEnd.Tests/TemSenhaTests.cs`.

⚠️ **`AcessoActyonWeb` não tem `SenhaCifrada`.** O acesso ao Actyon Web é por URL/sessão, sem credencial. Isso é regra de domínio, verificada por reflexão no mesmo teste. Ela **tem** `Usuario` (veja a exceção de mapeamento acima).

✅ **Divergência entre migration EF, script e banco — encerrada.** Este texto antes afirmava que `tbacessoactyonweb` tinha **duas** colunas órfãs sem prefixo (`SenhaCifrada` e `Usuario`) não mapeadas pela entidade. Isso estava **meio errado**:

- **`SenhaCifrada` era órfã de verdade e foi removida.** A migration [`20260929013916_AcessosEmpresas_DropColunasOrfasActyonWeb`](../../../../backend/Central_BackEnd/Migrations/20260929013916_AcessosEmpresas_DropColunasOrfasActyonWeb.cs) faz o `DROP COLUMN` (o `Down` recria como `nvarchar(400)`) e está **aplicada** em `dbBUSINESS_HML`, com script em [Sql/AcessosEmpresas_DropColunasOrfasActyonWeb.sql](../../../../backend/Central_BackEnd/Migrations/Sql/AcessosEmpresas_DropColunasOrfasActyonWeb.sql). O snapshot foi regenerado e está em sincronia com o banco: nele só restam as 4 `*_SenhaCifrada` prefixadas.
- **`Usuario` nunca foi órfã** — é legítima e mapeada, por herança da base sem `[Column]`. Foi preservada, e o script `AcessosEmpresas_Idempotente.sql` passou a criá-la como `Usuario nvarchar(100) NULL` com comentário explicando a ausência de prefixo. Reexecutar o script é no-op.
- **`CAO_Cnpj` não existe mais**: a coluna (e a da fantasia) foi dropada pela `20261002161552_RemoveIdentidadeLocalAcessos`, aplicada em `dbBUSINESS_HML` em 2026-10-02 (ver a seção de migrations abaixo).

`tbacessoactyonweb` ficou com **11 colunas**: `AWB_Id`, `AWB_ClienteAcessoId`, `AWB_Observacao`, `AWB_Ativo`, `AWB_DataAlteracao`, `AWB_UsuarioAlteracao`, `AWB_Url`, `AWB_Porta`, `AWB_Stack`, `AWB_Ambiente` e `Usuario`.

