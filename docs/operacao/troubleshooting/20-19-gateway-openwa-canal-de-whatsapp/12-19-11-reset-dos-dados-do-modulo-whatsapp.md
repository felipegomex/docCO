---
id: 20-19-gateway-openwa-canal-de-whatsapp.12
titulo: "19.11 Reset dos dados do módulo WhatsApp"
tipo: operacao
area: diagnostico
sistema: central
tags: [whatsapp, openwa, gateway, erro]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 1209
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.11 Reset dos dados do módulo WhatsApp

**Quando usar.** Para zerar o **histórico operacional** do canal (mensagens, atendimentos, eventos, contatos, grupos e números) sem tocar no schema, nas configurações nem nos catálogos — por exemplo, ao encerrar um piloto e começar a operação limpa, ou ao descartar dados de teste de um ambiente. **Não** use para corrigir defeito de schema: para isso existem os scripts aditivos de `Migrations/Sql`.

**Script:** [`WhatsApp_Reset_Dados_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Reset_Dados_Idempotente.sql). Não é migration EF: **não carimba `__EFMigrationsHistory`** e é executado **manualmente** no banco de cada ambiente. É um **lote único** (sem `GO`, porque o `SqlClient` não entende o separador), idempotente, com `SET XACT_ABORT ON` + `BEGIN TRAN` + `TRY/CATCH` (erro = `ROLLBACK` total), `IF OBJECT_ID` em cada passo e contagens antes/depois (a contagem `-1` significa que a tabela não existe naquela base).

**Parâmetros no topo do arquivo:**

| Parâmetro | Padrão | Efeito |
|---|---:|---|
| `@Executar` | `0` | `0` = apenas contagens, **não altera nada**; `1` = executa o reset |
| `@ApagarNumeros` | `1` | `1` = apaga `tbwhatsapp_numero`; `0` = preserva os números cadastrados |
| `@ResetarIdentidades` | `1` | `1` = `DBCC CHECKIDENT (RESEED, 0)` nas tabelas apagadas (próximo `INSERT` volta a 1); `0` = não mexe |

**Ordem de exclusão (FK-safe):** `tbwhatsapp_webhook_fila` → `tbwhatsapp_mensagem` → `tbwhatsapp_evento` → `tbwhatsapp_atendimento_etiqueta` → `tbwhatsapp_atendimento_tag` (legado, só se existir) → `UPDATE tbwhatsapp_conversa SET WAT_AtendimentoId = NULL` (quebra a FK circular conversa ↔ atendimento) → `tbwhatsapp_atendimento` → `tbwhatsapp_conversa` → `tbwhatsapp_participante` → `tbwhatsapp_grupo` → `tbwhatsapp_contato` → `tbwhatsapp_numero` (só com `@ApagarNumeros = 1`) → `tbwhatsapp_auditoria` → `tbwhatsapp_retencao_execucao`.

**Preserva:** `tbwhatsapp_sla_config`, `tbwhatsapp_sla_horario`, `tbwhatsapp_sla_historico`, `tbwhatsapp_retencao_config`, `tbwhatsapp_parametro`, `tbwhatsapp_etiqueta` (catálogo), `tbwhatsapp_resposta_rapida`, `tbwhatsapp_arquivamento_config` e `tbwhatsapp_ia_config`.

O reseed é condicionado a `sys.identity_columns`: `tbwhatsapp_atendimento_etiqueta` tem **PK composta, sem IDENTITY**, e por isso não recebe `DBCC CHECKIDENT`.

**Como rodar:**

1. Faça backup `COPY_ONLY` com `CHECKSUM` do banco do ambiente e valide com `RESTORE VERIFYONLY`. **Não** rode o reset sem backup conferido.
2. Primeiro passe com `@Executar = 0` (padrão) e confira as contagens e a fotografia das tabelas preservadas.
3. Troque `@Executar` para `1` e rode novamente. Exemplo (credenciais **fora** da linha de comando):

```powershell
sqlcmd -S 192.168.2.154 -d dbCENTRAL_OPERACAO -b -i backend\Central_BackEnd\Migrations\Sql\WhatsApp_Reset_Dados_Idempotente.sql
```

4. Confira as contagens **depois** (todas as tabelas apagadas em `0`) e a configuração preservada. Repetir o script na base já limpa é *no-op*.

⚠️ O antigo `WhatsApp_Limpeza_Dados.sql` está **deprecado**: não limpava `tbwhatsapp_grupo`, `tbwhatsapp_participante`, `tbwhatsapp_auditoria` nem `tbwhatsapp_retencao_execucao`, **apagava** `tbwhatsapp_arquivamento_config` e não tinha transação única. Use somente o script novo.

**Execução real (2026-10-27).** Duas passagens no mesmo dia. Na segunda (23:29), o reset foi aplicado em `dbCENTRAL_OPERACAO` (PROD) e `dbBUSINESS_HML` (HML), no SQL Server `192.168.2.154`, após backup `COPY_ONLY` + `CHECKSUM` com `RESTORE VERIFYONLY` nos dois bancos — `dbCENTRAL_OPERACAO_pre_reset_whatsapp_20261027-232949.bak` e `dbBUSINESS_HML_pre_reset_whatsapp_20261027-232955.bak`, em `C:\Program Files\Microsoft SQL Server\MSSQL14.MSSQLSERVER\MSSQL\Backup\`. Resultado:

- **PROD:** as 13 tabelas operacionais em `0`, `tbwhatsapp_numero` vazia, identidades reseed para `0`; SLA intacta (metas 8/240 min; horários seg–sex 07:00–18:00), `tbwhatsapp_etiqueta` com 3 registros e `tbwhatsapp_sla_historico` com 2.
- **HML:** as 13 em `0` e numeração zerada; SLA intacta **com os valores próprios da HML** (30/240 min; 08:00–18:00), que não foram alterados.
- **Gateway OpenWA v0.25.0** (`http://192.168.2.253:2785`): `/api/health` respondeu `200`; a sessão `central-wa-2` foi removida (`DELETE` → **204**) e `GET /api/sessions` ficou em `[]`. A primeira passagem do dia (14:05) já havia encontrado o gateway vazio.
- **Não houve deploy de código** nesta tarefa; a mudança é só de dados.

