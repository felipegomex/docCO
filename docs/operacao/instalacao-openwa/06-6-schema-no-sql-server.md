---
id: instalacao-openwa.06
titulo: "6. Schema no SQL Server"
tipo: operacao
area: operacao
sistema: central
tags: [openwa, gateway, whatsapp, docker]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 834
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 6. Schema no SQL Server

O banco é **sempre** o SQL Server `192.168.2.154`.

**Arquivo:** [`backend/Central_BackEnd/Migrations/Sql/WhatsApp_Idempotente.sql`](../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Idempotente.sql) — cria as **7 tabelas** `tbwhatsapp_*` (número, contato, conversa, atendimento, mensagem, evento e fila de webhook), com FKs em `ON DELETE NO ACTION`, `CHECK` nos enums e índices (o **único** filtrado `(WAT_ChatId, WAT_ExternalId)` das mensagens e o **único** da chave de dedupe da fila). **100% aditivo** (a única exceção é o **bloco idempotente de ajuste de largura** de `WAT_Status`, §6.3) e **não** carimba `__EFMigrationsHistory`.

### 6.1 Estado (2026-10-21)

| Item | Valor |
|---|---|
| Base | `dbBUSINESS_HML` no SQL Server `192.168.2.154` (mesmo servidor do `.254`, base diferente) |
| Resultado | 7 tabelas `tbwhatsapp_*`, 11 FKs, 7 `CHECK`, 89 colunas |
| Idempotência | **reaplicação é no-op** (conferida) |
| Backup | **sem backup**, por decisão explícita do dono do projeto |
| Execução | de dentro da rede, lendo a connection string do `appsettings.json` do próprio servidor, por `System.Data.SqlClient` |

### 6.2 Aplicar o script `.sql` em lotes — o `GO` não é SQL

O `SqlClient` **não entende** o separador `GO`: enviar o arquivo inteiro devolve *"Sintaxe incorreta próxima a 'GO'"*. O script precisa ser quebrado em lotes nos separadores `GO` e cada lote executado separadamente. Diagnóstico e alternativas em [`troubleshooting.md`](../troubleshooting.md) §5.1.

⚠️ **`deploy.ps1` não roda `Migrate()` nem `dotnet ef`.** Schema e binário podem divergir silenciosamente: o script do banco vai **antes** do publish.

### 6.3 O bloco de ajuste de largura (`WAT_Status`)

O primeiro deploy do script criou `tbwhatsapp_numero.WAT_Status` como **`varchar(20)`**. O nome do evento do gateway (`session.authenticated`, **21 caracteres**) não cabia: gravar o nome do evento no status estourava o INSERT/UPDATE com o **erro 8152** ("dados de cadeia ou binários seriam truncados"), a linha da fila ficava presa em "processando" e o **telefone do número nunca era gravado**.

Correção: a coluna passa a **`varchar(50)`** e o código grava sempre um **status canônico**. O script ganhou um **bloco idempotente de ajuste**, que alarga a coluna em base já instalada sem recriar a tabela:

```sql
IF EXISTS (SELECT 1 FROM sys.columns c JOIN sys.types t ON t.user_type_id = c.user_type_id
           WHERE c.object_id = OBJECT_ID('tbwhatsapp_numero') AND c.name = 'WAT_Status'
                 AND t.name IN ('varchar', 'nvarchar') AND c.max_length < 100)
BEGIN
    ALTER TABLE tbwhatsapp_numero ALTER COLUMN WAT_Status varchar(50) NULL;
    PRINT 'tbwhatsapp_numero.WAT_Status alargado para varchar(50).';
END
```

⚠️ **O script precisa ser reaplicado** quando houver mudança de largura: publicar o código novo **sem** reaplicar o script mantém o `varchar(20)` e o erro 8152 volta. Ver [`deploy.md`](../deploy.md) §7 e [`troubleshooting.md`](../troubleshooting.md) §19.10.1.

