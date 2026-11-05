---
id: 20-19-gateway-openwa-canal-de-whatsapp.14
titulo: "19.13 Protocolo/foto/telefone do contato — coluna ausente após publicar as Fases 0/1"
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
tokens_estimados: 667
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.13 Protocolo/foto/telefone do contato — coluna ausente após publicar as Fases 0/1

**Sintoma.** O backend novo (com protocolo e telefone do contato) sobe contra uma base que ainda não tem o script da fase: `GET /api/v1/whatsapp/atendimentos` ou o detalhe respondem **500** com `Invalid column name 'WAT_Protocolo'`; ao abrir contato/ficha, o erro muda para `WAT_FotoUrl`, `WAT_FotoAtualizadaEm` ou `WAT_TelefoneResolvido`.

**Causa.** O `Atendimento.Protocolo` é mapeado como coluna computada e os três campos do contato são colunas reais; o EF os inclui no `SELECT`. **Não há migration EF** para o módulo — o DDL vive em `Migrations/Sql/WhatsApp_Fase8_Idempotente.sql`. O `deploy.ps1` não aplica schema, então o binário ficou à frente do banco.

**Correção.** Rode [`WhatsApp_Fase8_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/WhatsApp_Fase8_Idempotente.sql) no banco do ambiente **antes** do publish (a ordem do `AGENTS.md` é SQL do commit → deploy). O script é aditivo e idempotente: cria `WAT_Protocolo` computada PERSISTED + índice único, `WAT_FotoUrl`/`WAT_FotoAtualizadaEm`/`WAT_TelefoneResolvido`, faz o backfill de `WAT_Numero` sem JID e insere `atendimento.finalizacaoAutomaticaMinutos` (15) e `atendimento.fotoTtlHoras` (24). Reaplicar é no-op. Sem os parâmetros, o módulo usa o default e nada quebra; sem as colunas, a leitura quebra. ✅ **Aplicado 2× em HML (`dbBUSINESS_HML`) e 2× em PROD (`dbCENTRAL_OPERACAO`) em 2026-10-27**, com idempotência comprovada na segunda execução e backups `dbBUSINESS_HML_pre_fase8_whatsapp_20261027-232815.bak` e `dbCENTRAL_OPERACAO_pre_fase8_whatsapp_20261027-232853.bak`.

⚠️ **Não confunda com o reset.** `WhatsApp_Reset_Dados_Idempotente.sql` apaga **dados** e não cria schema — ele não resolve `Invalid column name`.

**Cobertura.** O protocolo, a busca e o telefone são exercitados por WA-84/WA-85 em [`qa-historias.md`](../../../referencia/qa-historias.md) e por `WhatsAppProtocoloTests` (3 casos). A foto tem o roteiro WA-87, coberto por `WhatsAppFotoContatoTests` (3 casos) e pelo spec `contato-avatar.component.spec.ts` (3 casos, **não executado** nesta medição); a finalização automática tem o roteiro WA-86, coberto por `WhatsAppFinalizacaoAutomaticaTests` (4 casos). As métricas por etiqueta (WA-88) têm `WhatsAppMetricasEtiquetaTests` (2 casos).

