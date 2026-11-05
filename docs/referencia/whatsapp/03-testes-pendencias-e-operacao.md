---
id: whatsapp.03
titulo: "Atendimento WhatsApp — testes, pendências e execução operacional"
tipo: referencia
area: atendimento
sistema: central
tags: [whatsapp, atendimento, testes, pendencias, deploy, operacao]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd.Tests
  - frontend/src/app/features/whatsapp
  - scripts
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp
tokens_estimados: 1980
ficha: docs/modulos/whatsapp.md
---
> Parte de [Atendimento WhatsApp — implementação, contratos e operação](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## Testes

- Fases 6/7: `WhatsAppAtendimentoTests.cs` (mp4/webm/gif, 10 MB, documento
  recusado, contato: feliz/409/falha), `WhatsAppVCardTests.cs` e
  `WhatsAppContratoGatewayTests.cs` (`send-video`, `send-contact`, fallback
  404, 500 sem fallback). Suíte `WhatsApp*` + specs do composer/página/balão.
- Fase 8: `WhatsAppAtendimentoTests.cs` (persistência, dedupe, remoção, `502`,
  flag desligada, sem `ExternalId`), `WhatsAppWebhookTests.cs` (evento novo,
  dedupe, remoção) e `whatsapp-message.component.spec.ts` (6 specs: menu,
  chip, flag, `externalId`, nota interna).
- Rodada de 2026-10-30 (telefone/`@lid`, menções, prefixo, tempo real, busca e
  envio): **473 testes WhatsApp** verdes no backend (prefixo `*NOME:*`, menção
  por nome, `FormatarExibicao`, resolução `@lid` sob demanda) e **266 specs** no
  frontend (composer, realtime, Kanban).
- Rodada de 2026-11-01 (finalizar sem mensagem, prefixo no encerramento,
  automação + arquivamento, FCR e configuração da janela): **483 testes
  WhatsApp** verdes — `WhatsAppAtendimentoTests.cs`,
  `WhatsAppFinalizacaoAutomaticaTests.cs` e `WhatsAppConfiguracaoTests.cs`.
- Rodada de 2026-11-03 (contato só no vínculo): **1075 testes** no backend e
  **258 specs** no frontend. Em `WhatsAppContatoTests.cs`:
  `Entrada_de_numero_desconhecido_nao_cadastra_contato`; em
  `WhatsAppGrupoAtendimentoTests.Solicitar_privado_abre_conversa_sem_cadastrar_contato`;
  a criação pelo vínculo fica em `WhatsAppContatoVinculoTests.cs` e os
  fallbacks de nome/telefone em `whatsapp.utils.spec.ts` (`nomeOuTelefone`).
- Rodada de 2026-11-03 (Contatos e iniciar conversa): `scripts/validate.ps1`
  verde com **1083 testes** no backend e **275 specs** no frontend.
  `WhatsAppIniciarConversaTests.cs` cobre conversa existente, número novo sem
  contato, reaproveitamento, 409, 400 (chat/telefone), 404 e auditoria
  `iniciar_conversa`; no frontend, `whatsapp-contatos-painel.component.spec.ts`,
  `iniciar-conversa-modal.component.spec.ts` e
  `atendimento-modal-responsabilidade.component.spec.ts` cobrem painel, modal
  e conflito de responsabilidade.
- QA: `docs/referencia/qa/17-12-atendimento-whatsapp/INDEX.md` (WA-01–125).

## Pendências

- **Contatos e iniciar conversa:** o fluxo está coberto por testes de service e
  de componente, mas falta exercitá-lo no navegador com sessão real (200 →
  abre o atendimento; 409 → modal de responsabilidade; 400 no campo de
  telefone; 404 com número/sessão indisponível) e confirmar que o painel nem o
  gateway criam contato. Sem schema novo e sem envio de mensagem ao iniciar.
- **Reset de 2026-10-30:** 13 tabelas operacionais zeradas em PROD e HML e
  gateway **sem sessão** (`GET /api/sessions` = `[]`); pendente **reconectar um
  número** (sessão `central-wa-1`) e revalidar o fluxo de ponta a ponta. Não há
  mais backfill pendente: o telefone de `@lid` é resolvido **sob demanda** pelo
  `ContatoTelefoneResolver` na primeira leitura da ficha — falta apenas exercitar
  a resolução contra uma sessão real após a reconexão.
- **Tempo real em PROD:** o hub passou a derivar da `apiBaseUrl` quando o
  `whatsappHubUrl` preservado é de outro ambiente; falta confirmar no navegador
  de PROD que a conexão usa `:1009` do próprio servidor (smoke não cobre o hub).
- Cobertura de tela parcial; backend coberto por `PermissoesContratosTests`.
- Fases 6/7: vídeo e GIF (D2) validados em 2026-10-30; faltam caminho A/B do
  contato e teto real do gateway (D5).
- Fase 8: **medição formal adiada** (bloco A, decisão do usuário em
  2026-10-30) — `messages/react`, `GET .../reactions` e o evento
  `message.reaction` seguem **NÃO TESTADOS**; há só evidência informal (relato
  do usuário), sem status HTTP nem payload, e **nada pode ser escrito como
  "suportado"**. SQL **aplicado** nos dois bancos de homologação e flag
  habilitada por configuração; pendentes o **modo limitado (C3)** se o payload
  vier fora do formato, o caminho A/B do contato e a versão do gateway por
  `/health`.

## Execução operacional

- **Deploy de 2026-11-03 (PROD, commit `14fc6e3`):** `scripts/validate.ps1`
  verde (backend **1075** testes, frontend **261** specs, docs 0 erros); pacote
  **PROD** gerado no próprio commit (`BUILD_INFO` confere com o HEAD);
  preflight OK em `dbCENTRAL_OPERACAO` (`192.168.2.154`); publicado no IIS PROD
  (`192.168.2.254`) com backup em
  `\\192.168.2.254\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261103-163440`;
  `appsettings.json`, `runtime-config.json` e as variáveis `WhatsApp__*` do
  `web.config` preservados. Smoke **5/5** (frontend `200`, swagger `200`,
  `runtime-config` para `http://192.168.2.254:1009/api/v1`, login fictício
  `401` e webhook `401`). Inclui `c5520f2` (telefone/foto do `@lid` pelo campo
  `phone` do OpenWA 0.25) e `14fc6e3` (abordagem anterior de contato pela
  conversa, substituída pelo vínculo em Admin > Contatos).

- **Deploy de 2026-10-30 (PROD, commit `6d2bbc1`):** `scripts/validate.ps1`
  verde (backend **1060** testes, frontend **266** specs, docs 0 erros); pacote
  HML reaproveitado sem rebuild; preflight OK em `dbCENTRAL_OPERACAO`
  (`192.168.2.154`); publicado no IIS PROD (`192.168.2.254`) com backup em
  `\\192.168.2.254\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261030-172340`
  e `web.config` preservado (`web.config.pre-deploy-20261030-172349`). Smoke
  **5/5** (frontend `200`, swagger `200`, `runtime-config` para
  `http://192.168.2.254:1009/api/v1`, login fictício `401` e webhook `401`).
  **HML (`192.168.2.130`) pendente.** O banco de PROD é
  `dbCENTRAL_OPERACAO` — a tabela de ambientes citava `dbBUSINESS` e foi
  corrigida.
- **Gate e deploy:** commit `16ae43d` na branch `developer` (push OK), com
  `scripts/validate.ps1` verde antes da publicação — backend **1052** testes,
  frontend **264** e docs OK. Publicado em **HML** (`192.168.2.130`) com pacote
  e publicação OK, `BUILD_INFO` = `16ae43d` e backup IIS em
  `\\192.168.2.130\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261030-153136`;
  em **PROD** (`192.168.2.254`) com preflight OK em `dbCENTRAL_OPERACAO`,
  publicação OK e backup IIS em
  `\\192.168.2.254\c$\Users\JCASRV-SUP\Documents\Backup_IIS\20261030-153456`.
- **Smoke OK nos dois ambientes:** frontend `200`, swagger `200`,
  `runtime-config` com a API do próprio servidor (`http://192.168.2.130:1009/api/v1`
  em HML e `http://192.168.2.254:1009/api/v1` em PROD), login fictício `401` e
  webhook `401`.
- **Reset WhatsApp:** `scripts/whatsapp-reset.ps1 -Ambiente AMBOS -Confirmar`
  removeu a sessão OpenWA `central-wa-1`
  (`6cd53a88-b19c-4102-8b88-94e5451acbbd`, `DELETE` com `204`) e o gateway ficou
  **sem sessão** (`GET /api/sessions` = `[]`); zerou **13 tabelas operacionais**
  em `dbCENTRAL_OPERACAO` (PROD) e `dbBUSINESS_HML` (HML), incluindo
  `tbwhatsapp_contato`. Backups verificados com `RESTORE VERIFYONLY` OK:
  `dbCENTRAL_OPERACAO_pre_reset_whatsapp_20261030-153628.bak` e
  `dbBUSINESS_HML_pre_reset_whatsapp_20261030-153637.bak`.
- **SLA preservado** no reset: PROD **8/240 min** e HML **30/240 min**, com
  **5 faixas de horário** em cada banco.
- **Backfill de `@lid` apagado:** os **43** telefones resolvidos antes da
  limpeza (ver [02](./02-regras-e-contratos-do-modulo.md)) **não estão gravados**
  hoje; o módulo está zerado e aguarda a reconexão de um número.
