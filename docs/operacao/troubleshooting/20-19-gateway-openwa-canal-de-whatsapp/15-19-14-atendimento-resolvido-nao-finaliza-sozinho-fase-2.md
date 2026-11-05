---
id: 20-19-gateway-openwa-canal-de-whatsapp.15
titulo: "19.14 Atendimento Resolvido não finaliza sozinho (Fase 2)"
tipo: operacao
area: diagnostico
sistema: central
tags: [whatsapp, openwa, gateway, erro]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 579
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.14 Atendimento Resolvido não finaliza sozinho (Fase 2)

**Sintoma.** Cards parados em **Resolvido** passam da janela configurada e continuam na coluna; nenhum evento `finalizacao_automatica`/`arquivamento` aparece na timeline e o Kanban não recebe atualização.

**Causa provável (na ordem em que se confere).**

1. **O parâmetro está desligado.** `atendimento.finalizacaoAutomaticaMinutos` **`<= 0`** faz a rodada retornar sem tocar em nenhum card. Confira em `/whatsapp/configuracao` (card **"Automação do atendimento"**) ou em `tbwhatsapp_parametro`. O `WhatsAppParametroService` cacheia por **30 s**, então o efeito de uma alteração não é imediato.
2. **A janela ainda não venceu.** O critério é `WAT_ResolvidoEm <= agora - minutos`; o job roda a cada **1 min** e a seleção é ordenada por `WAT_ResolvidoEm`, em lotes de **200**. Um backlog grande leva mais de uma rodada.
3. **É grupo.** A regra exige `GrupoId` **nulo** e `ChatId` **fora** de `@g.us`: grupo nunca finaliza sozinho (decisão do plano).
4. **O card não está Resolvido** ou está sem `WAT_ResolvidoEm` (dado legado). O `UPDATE` é condicional (`Status = 3` + corte): se o status mudou, afeta 0 linhas e nada é gravado — inclusive o evento.
5. **O job não está registrado.** `Program.cs` precisa de `AddScoped<WhatsAppFinalizacaoAutomaticaService>()` **e** `AddHostedService<WhatsAppFinalizacaoAutomaticaJob>()`; erro em uma rodada é logado como `Erro na rodada da finalização automática do WhatsApp.` e o loop continua.

**Diagnóstico.** Conferir o parâmetro, o `WAT_Status`/`WAT_ResolvidoEm` do card, o log do backend (`Finalização automática: N atendimento(s) finalizado(s) e arquivado(s) após X min.` sai só quando N > 0) e a tabela `tbwhatsapp_evento` pelos tipos `finalizacao_automatica` e `arquivamento`.

**Importante.** A rodada grava **`Arquivado`** (não `Finalizado`): o card sai do Kanban operacional e continua pesquisável com "incluir arquivados". A automação **não** é desfeita por falha de SignalR: a publicação de `atendimento_atualizado`/`fila_alterada` é **depois do commit** e best-effort. Se o card está arquivado e a tela não mudou, o problema é notificação/polling, não a regra. Não há botão de execução manual — a alavanca é a janela (tela de configuração ou parâmetro no banco).

