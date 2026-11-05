---
id: sec.16-15-checklist-de-conclusao
titulo: "15. Checklist de conclusão"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 2529
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 15. Checklist de conclusão

### Antes do package

- [ ] Ambiente local testado.
- [ ] Primeiro validate verde; a compilação Debug não foi tratada como prova do Release.
- [ ] `validate.ps1` verde nos **7/7**, incluindo `dotnet test` e `ng test`.
- [ ] Commit/push concluído no fluxo normal.
- [ ] Working tree limpa. Se publicar de árvore suja em emergência, registre a divergência entre `BUILD_INFO.GitCommit` e o binário e **não** trate o publish como reproduzível.
- [ ] SQL do commit aplicado — incluindo os scripts do RBAC quando o código novo mapear `CARGO_Nivel`/`CARGO_FuncaoId` (o preflight cobre a **existência** das colunas, mas não o **tipo**) e o **aditivo do vínculo número → setor** quando o commit mexer em `tbwhatsapp_numero` (a consolidação `NOT NULL`/`DROP WAT_Fila` roda **depois** do publish).
- [ ] Ambiente de destino definido (`-Ambiente HML|PROD`); em PROD, `-Backup 1` e confirmação digitando `PROD`.
- [ ] No primeiro deploy do ambiente, `appsettings.json` e `runtime-config.json` (com a `apiBaseUrl` do ambiente) já criados no servidor — o pre-flight aborta sem eles.
- [ ] Migration revisada/aplicada quando necessária. Se o commit mexeu no Dashboard Integrado, confirme `__EFMigrationsHistory` e `COL_LENGTH('tblead','LEAD_ContatoVisualizado')` no banco alvo.
- [ ] Se o commit mexeu no Acessos Empresas, confirme as 6 tabelas do módulo (a 7ª, `tbgrupoacesso`, deve **não existir** após `AcessosGrupoClienteErp`) e as 3 colunas `AAC_*` no banco alvo **antes** de publicar, e confira se `eventosAuditoriaPerdidos` está em 0 em `/api/v1/acessos/estado-cifra`.
- [ ] **Se o commit mexeu no eixo grupos/ERP do Acessos (busca de empresa, `tbcliente_grupo`)** e o banco alvo ainda não tem a migration: rode `Migrations/Sql/AcessosGrupoClienteErp_Idempotente.sql` **imediatamente antes** do publish (janela de incompatibilidade: código antigo grava ids de `tbgrupoacesso` em `CAO_GrupoId`); depois do smoke, confirme `OBJECT_ID('tbgrupoacesso') IS NULL`, `CAO_ClienteId` existente e a migration carimbada em `__EFMigrationsHistory`. Pré-requisito: `AcessosEmpresas_Idempotente.sql`. Em `dbBUSINESS_HML` já foi feito em 2026-09-30.
- [ ] Se o commit mexeu no Dashboard Integrado, o QA manual cobriu `/dashboard/comercial` (inclusive o toggle "Viu/Não viu"), `/dashboard/administrativo` e, quando houver atraso, `/dashboard/financeiro` — o smoke não cobre nenhuma dessas rotas.
- [ ] Se o commit mexeu no Acessos Empresas, o QA manual cobriu `/acessos/clientes` (filtros, paginação, gate de senha, as 5 abas, uma revelação em RDP/VPN e o auto-esconder de 30 s) — o smoke não cobre nenhuma dessas rotas.
- [ ] Se o commit mexeu no Kanban Geral, confirme `tbbloqueio` e as colunas `CLK_StatusCard` no banco alvo **antes** de publicar, e faça QA manual de `/kanban` (pill de função, Tarefas/Projetos/Ambos, card bloqueado, permissão read-only), `/kanban/gargalos` e `/kanban/admin/colunas`.
- [x] **Se o commit mexeu no eixo Setor → Função:** (a) **antes** do publish, rode `Migrations/Sql/SetorParaFuncao_Idempotente.sql` e valide a Fase 0; (b) publique e faça o smoke; (c) **só então** rode `Migrations/Sql/SetorParaFuncao_Consolidar_Idempotente.sql` e valide que sobrou zero coluna `*_SetorId` e que `20260929221556_SetorParaFuncao` está em `__EFMigrationsHistory`. Backup do banco antes da Fase 3. ⚠️ Não existe staging: `dbBUSINESS_HML` é produção. **→ Concluído em `dbBUSINESS_HML` em 2026-10-01** (Fase 3 + backups `bkp_20261001_pre_setorfuncao_*` + verificação de zero coluna `*_SetorId`).
- [x] **Se o commit mexeu nos quadros independentes por função (spec Seção 3):** rode `Migrations/Sql/ColunasPorFuncao_Backfill_Idempotente.sql` no banco alvo **antes** do publish (é idempotente; confira as 4 verificações do script: 0 funções sem coluna própria e 0 cards fora do quadro da própria função) e faça backup pré. Depois do publish, QA manual de `/kanban/admin/colunas` (grupos por função, criar/excluir/reordenar dentro do grupo) e `/implantacao/tarefas/novo` (dropdown só com colunas da função efetiva). ⚠️ Não existe staging: `dbBUSINESS_HML` é produção. **→ Concluído em `dbBUSINESS_HML` em 2026-10-02** (backup pré `dbBUSINESS_HML_pre-colunasfuncao_20261002.bak`, aplicação + reexecução de prova de idempotência, antes do deploy `63f0026`). **→ Modelo superado em 2026-10-20** (ver o item de colunas compartilhadas).
- [ ] **Se o commit mexeu no catálogo de colunas (modelo compartilhado, 2026-10-20):** faça **backup do banco** e rode `Migrations/Sql/ColunasCompartilhadas_Idempotente.sql` **ANTES do publish** (é idempotente; confira as 6 contagens do próprio script: 7 colunas de catálogo, 0 cópias por função, visões gravadas, visões com máscara, visões ocultas e 0 cards apontando para cópia). Publicar antes do script deixa o **quadro das funções vazio** — sintoma em [`09-TROUBLESHOOTING.md`](../troubleshooting.md). Depois do publish, QA manual de `/kanban/admin/colunas` (bloco **catálogo**: criar/editar/ativar-desativar/reordenar — **sem excluir**), de `/kanban/configurar` (visão do setor: interruptor de visível + máscara) e de `/kanban` (rótulo com máscara e dica "Catálogo: …"). **→ Concluído em `dbBUSINESS_HML` em 2026-10-20** (backup conferido com `RESTORE VERIFYONLY`; catálogo com 7 colunas, 0 cópias por função, 28 visões — 2 com máscara, da função 4 (Financeiro) — e 11 tarefas remapeadas).
- [ ] Se houve cards de RH ou Comercial no recorte antigo, confirme com o negócio a destino de cada um — eles ficam com `FUNCAO_ID NULL` e somem de qualquer recorte por função (e, com a pill "Todas" removida do quadro em 2026-10-01, **não aparecem em lugar nenhum** até ganharem função).
- [ ] Backup do banco definido quando houver mudança de schema.
- [ ] Saídas antigas limpas quando o package incremental puder reutilizar arquivos.

### Package/publicação

- [ ] Package Debug/Release distinction verificada: validate Debug e publish Release.
- [ ] `BUILD_INFO.GitCommit` igual ao HEAD; `Ambiente`/`Servidor` conferem com o destino; recibo em `deploy/history/`.
- [ ] Quatro artefatos obrigatórios presentes e sem resíduo de `/E`.
- [ ] Credencial obtida por `Get-Credential` ou secret manager (o script aborta sem `-SenhaRemota` explícito) e sessão de rede residual removida antes do publish (`net use \\<servidor>\C$ /delete /y`).
- [ ] Backup IIS criado com timestamp (obrigatório em PROD).
- [ ] `appsettings*.json` e `runtime-config.json` preservados; ausência no servidor aborta no pre-flight.
- [ ] Backend saiu de `app_offline.htm`.
- [ ] **Módulo WhatsApp ativo:** bloco `<environmentVariables>` (`WhatsApp__Habilitado`, `WhatsApp__BaseUrl`, `WhatsApp__WebhookUrl`, `WhatsApp__ApiKey`, `WhatsApp__WebhookSecret`) **reaplicado** no `web.config` do backend **depois** do publish (ele é sobrescrito a cada publicação) e backend **reciclado**. O bloco tem que estar **dentro** de `<aspNetCore>`, com `<environmentVariable name= value=>`.
- [ ] **Módulo WhatsApp ativo:** `Migrations/Sql/WhatsApp_Idempotente.sql` **reaplicado no banco do ambiente** quando o deploy trouxer mudança de schema (por exemplo, o alargamento de `WAT_Status` para `varchar(50)`, que tem bloco idempotente de ajuste) — **antes** do publish. O `deploy.ps1` não aplica DDL.
- [x] **Status `Visto` (F6):** `Migrations/Sql/WhatsApp_Visto_Idempotente.sql` aplicado no banco do ambiente (recria a `CHECK` do atendimento para **0..6**) **antes** do publish do backend que grava o status `6`, quando a base ainda tiver a `CHECK` 0..5. Aplicado em HML (`dbBUSINESS_HML`, 2026-10-26 19:24) e PROD (`dbCENTRAL_OPERACAO`, 2026-10-26 19:24), com backup `COPY_ONLY` e `RESTORE VERIFYONLY` antes.
- [x] **Protocolo/foto/telefone (Fases 0/1):** `Migrations/Sql/WhatsApp_Fase8_Idempotente.sql` aplicado no banco do ambiente **antes** do publish do backend que lê `WAT_Protocolo`/`WAT_FotoUrl`/`WAT_TelefoneResolvido`; sem ele, a leitura do atendimento falha com `Invalid column name`. **Aplicado 2× em HML e 2× em PROD em 2026-10-27** (segunda execução no-op), com backups `dbBUSINESS_HML_pre_fase8_whatsapp_20261027-232815.bak` e `dbCENTRAL_OPERACAO_pre_fase8_whatsapp_20261027-232853.bak`.
- [ ] Frontend publicado com fallback SPA.
- [ ] Regras IIS/reverse proxy para `/api/chat/*` e `/api/conhecimento/*` documentadas e testadas, ou indisponibilidade explicitamente aceita.
- [ ] Para Chat/Conhecimento, confirme aplicação de `20261021003310_ChatConhecimento`, segredos protegidos, callback OAuth, mapeamentos de workspace/revisores e provisionamento da VM de sincronização. Nesta revisão, esses pré-requisitos **não foram concluídos** e nenhuma publicação foi realizada.

### Validação

- [ ] `smoke.ps1` retornou 0 no servidor do ambiente — frontend 1010, Swagger 1009 e `runtime-config.json` com a `apiBaseUrl` do ambiente. ⚠️ Falha **só** no check “Swagger/API”, com `500` de corpo vazio em todas as rotas, costuma ser o **schema do `<environmentVariables>`** no `web.config` do backend — ver [`09-TROUBLESHOOTING.md`](../troubleshooting.md) §13.1.
- [ ] Se o módulo WhatsApp estiver ativo: `GET /api/v1/whatsapp/saude` responde com o gateway acessível (o `smoke.ps1` **não** cobre o módulo). Em 2026-10-27 o gateway ficou **sem sessões** após o reset dos dados — reconecte um número antes da validação manual.
- [ ] Login e refresh funcionais; logout foi verificado, incluindo o 401 conhecido.
- [ ] API autenticada representativa respondeu.
- [ ] Tela afetada foi validada.
- [ ] Integrações externas/banco foram testadas quando aplicáveis.
- [ ] `Secure=false` foi avaliado **e** a origem efetiva da chave JWT foi conferida no startup (`JWT: chave resolvida de {Origem}`), com `JWT_KEY` no ambiente da **máquina** (não do *application pool*).
- [ ] Nenhum segredo foi impresso ou publicado.

