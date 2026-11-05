---
id: 17-12-atendimento-whatsapp.indice
titulo: "12. Atendimento WhatsApp"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 500
ficha: docs/modulos/whatsapp.md
---

# 12. Atendimento WhatsApp

> Índice do documento dividido. Documento original: [17-12-atendimento-whatsapp.md](../17-12-atendimento-whatsapp.md).
> Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

> Parte de [docs/referencia/qa-historias.md](../../qa-historias.md).


> Módulo **implementado e publicado** em HML (`.130`) e PROD (`.254`): schema, gateway OpenWA v0.25.0 (VM Ubuntu `192.168.2.253`) e webhook validados ponta a ponta; a produção foi publicada em **2026-10-26** (smoke 5/5). Em **2026-10-27** HML e PROD receberam o pacote da **árvore de trabalho sobre o HEAD `b52d7bc` (sem commit)** — as features **WA-74 a WA-88** estão **no ar**, mas o binário publicado não corresponde a um commit. `WhatsApp_Fase8_Idempotente.sql` (WA-84/WA-85/WA-87) foi aplicado **2× nos dois bancos**, com idempotência comprovada e backups `dbBUSINESS_HML_pre_fase8_whatsapp_20261027-232815.bak` / `dbCENTRAL_OPERACAO_pre_fase8_whatsapp_20261027-232853.bak`; em PROD foi preciso aplicar também os scripts de permissões (login em 500) e o smoke final deu **5/5** nos dois ambientes. Depois do reset de dados de 2026-10-27, o gateway ficou **sem sessões**: reconecte um número antes da validação manual. **A Fase 8 (reações) está habilitada por configuração desde 2026-10-30** (`appsettings.json` com `ReacoesHabilitadas=true`; o modelo de ambiente novo nasce `false`; desligar = `WhatsApp__ReacoesHabilitadas=false`) e o script `WhatsApp_Fase9_Idempotente.sql` **foi aplicado nos dois bancos de homologação** (`dbCENTRAL_OPERACAO` e `dbBUSINESS_HML`) — os cenários WA-105 a WA-108 ficam executáveis. `messages/react` e `message.reaction` seguem **sem medição formal** (bloco A adiado): há apenas evidência informal, sem status HTTP nem payload. Os cenários **WA-118 a WA-128** cobrem o painel **Contatos**, o **iniciar conversa** e o **Vincular empresa pela conversa** (2026-11-03): contato salvo, número novo sem contato, reaproveitamento, 409, 400, 404, atendimento terminal, painel do Kanban, nome obrigatório no vínculo, o **Conversar** nos integrantes do grupo e o **filtro por empresa** do vínculo. Contrato do módulo em [`material-de-projeto/atendimento-whatsapp/README.md`](../../../../material-de-projeto/atendimento-whatsapp/README.md). Os cenários **WA-109 a WA-112** cobrem a rodada de **2026-10-30** (telefone/`@lid` sob demanda, menções por nome, prefixo `*NOME:*`, tempo real e composer). Os cenários **WA-113 a WA-117** cobrem o **contato só no vínculo** (2026-11-03, atualizado): entrada desconhecida não cria contato, o vínculo em Admin > Contatos cria o contato e assume o histórico, grupo é recusado e os integrantes mostram nome/foto.

## Partes

- [WA-01 — Receber uma mensagem e abrir o atendimento (+9)](./01-wa-01-receber-uma-mensagem-e-abrir-o-atendimento.md)
- [WA-09 — Administrar os números conectados (+6)](./02-wa-09-administrar-os-numeros-conectados.md)
- [WA-16 — Gravar o status canônico da sessão sem estourar a coluna (+8)](./03-wa-16-gravar-o-status-canonico-da-sessao-sem-estourar-a-colu.md)
- [WA-38 — Vincular o contato à empresa do ERP (Fase 01) (+7)](./04-wa-38-vincular-o-contato-a-empresa-do-erp-fase-01.md)
- [WA-46 — Alternar investigação sem mudar status, responsável ou SLA (Fase 06) (+9)](./05-wa-46-alternar-investigacao-sem-mudar-status-responsavel-ou.md)
- [WA-56 — SLA crítico destacado e filtrado por dimensão (Fase 07) (+6)](./06-wa-56-sla-critico-destacado-e-filtrado-por-dimensao-fase-07.md)
- [WA-63 — Atualização em tempo real sem esperar o polling (Fase 09) (+6)](./07-wa-63-atualizacao-em-tempo-real-sem-esperar-o-polling-fase-0.md)
- [WA-70 — Navegação inferior só no mobile (Fase 10) (+8)](./08-wa-70-navegacao-inferior-so-no-mobile-fase-10.md)
- [WA-79 — Nome do grupo como título do card no Kanban (Feature 2) (+6)](./09-wa-79-nome-do-grupo-como-titulo-do-card-no-kanban-feature-2.md)
- [WA-86 — Resolvido finaliza e arquiva sozinho após a janela, exceto grupo (Fase 2) (+3)](./10-wa-86-resolvido-finaliza-sozinho-apos-a-janela-exceto-grupo.md)
- [WA-90 — UX da tela de números: filtros na URL, modal, menu de ações e exclusão em duas etapas](./11-wa-90-ux-da-tela-de-numeros-filtros-na-url-modal-menu-de-aco.md)
- [WA-91 — Autoria no balão, modal de responsabilidade, negrito e Ctrl+V de imagem (Fases 1–4 do plano 2026-10) (+3)](./12-wa-91-autoria-modal-de-responsabilidade-negrito-e-ctrl-v.md)
- [WA-95 — Lightbox de mídia e busca server-side (Fases 5 e 9 do plano 2026-10) (+1)](./13-wa-95-lightbox-de-midia-e-busca-server-side-fases-5-e-9.md)
- [WA-97 — Envio de vídeo, GIF e cartão de contato (Fases 6 e 7 do plano 2026-10) (+4)](./14-wa-97-envio-de-video-gif-e-contato-fases-6-e-7-do-plano.md)
- [WA-101 — Validações, 409, progresso e nota interna no envio (Fases 6 e 7) (+4)](./15-wa-101-validacoes-409-progresso-e-nota-interna-fases-6-e-7.md)
- [WA-105 — Reações da Fase 8: flag, evidência e modo limitado (+3)](./16-wa-105-reacoes-atras-da-flag-fase-8.md)
- [WA-109 — Telefone/@lid, menções, prefixo, tempo real e composer (rodada 2026-10-30) (+4)](./17-wa-109-telefone-lid-mencoes-prefixo-tempo-real-e-nova-conversa.md)
- [WA-113 — Contato no vínculo: conversa e Admin (rodada 2026-11-03) (+4)](./18-wa-113-cadastro-manual-de-contato.md)
- [WA-118 — Contatos, iniciar conversa e vínculo pela conversa (rodada 2026-11-03) (+10)](./19-wa-118-contatos-e-iniciar-conversa.md)
