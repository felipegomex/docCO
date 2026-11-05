# Ideias, Melhorias e Erros — pendências para implantar

Data de origem: 2026-11-02 (revisado em 2026-11-03)
Status: pendente. Itens já implementados foram removidos deste plano.
Ficha do módulo: [modulos/ideias.md](../../modulos/ideias.md)

## Decisões já tomadas (vigentes)

1. Justificativa obrigatória na rejeição e na reabertura (efetivação na reabertura também).
2. Leitura sem permissão responde 404; ação sem permissão responde 403.
3. Ciclo pós-aprovação: Sentido 1. Após aprovação pelo gestor da Ciaa, a tarefa é submetida a outro colaborador do CIAA até finalizar, e segue para o Kanban. Sentido 2 descartado.
4. Exclusão: sem delete físico; apenas arquivamento.

## Pendências

### P1 — Atribuição e revisão no Kanban (Sentido 1)

- Hoje a aprovação aceita `ResponsavelId` e cria a tarefa já atribuída; não há reatribuição depois de aprovada.
- Falta definir e implementar como a tarefa é submetida a outro colaborador do CIAA após a aprovação, até a finalização.
- Verificar no Kanban como fica a revisão (quem vê, quem move, quem finaliza).
- Pontos de partida: `backend/Central_BackEnd/Services/Ideias/IdeiaKanbanService.cs#AprovarAsync`; `backend/Central_BackEnd/Services/Tarefas` (a confirmar).
- Antes de implementar: confirmar com o produto a regra de finalização e se a reatribuição mexe no vínculo `IDE_TarefaVinculadaId`.

### P2 — Conversa com quem abriu a ideia (UX)

- Tornar a relação comentário/conversa com o autor mais intuitiva e resumida.
- Pontos de partida: `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.html` (timeline e caixa de mensagem).

### P3 — Ações F6/F7/F14 indisponíveis

- Recomendação: exibir rótulo visível "Disponível em breve" nas ações ainda não implementadas.
- Hoje o motivo aparece apenas no `title` (`detalhe-ideia.component.html`, `b.motivoIndisponivel`).

### P4 — Backend decide as ações (completar)

- O endpoint `GET /ideias/minhas-permissoes` existe, mas a regra ainda é espelhada no cliente.
- Meta: a UI consome só o contrato do backend, sem replicar a regra.
- Pontos de partida: `backend/Central_BackEnd/Services/Ideias/IdeiasPermissaoService.cs`; `frontend/src/app/features/ideias/pages/detalhe/detalhe-ideia.component.ts`.

## Riscos

- Divergência de RBAC do gestor geral em relação ao ADR 004. Verificar antes de implementar P1 e P4.

## Documentos a atualizar ao implantar

- `docs/modulos/ideias.md` (regra de submissão pós-aprovação, seção 7 e pendências, seção 14).
- `docs/referencia/qa/15-10-ideias-melhorias-e-erros.md` (cenários novos).
- Documento do Kanban, se a reatribuição alterar o quadro.
