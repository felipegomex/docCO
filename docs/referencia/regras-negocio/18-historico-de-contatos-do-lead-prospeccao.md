---
id: regras-negocio.18
titulo: "Histórico de contatos do lead (prospecção)"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 455
ficha: docs/modulos/dashboard.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Dashboard Integrado](../../modulos/dashboard.md).

## Histórico de contatos do lead (prospecção)

**Situação: implementado; registro manual, automação de `UltimoContato` e badge visual.**

- Tipos fixos de contato (espelha `TiposContatoLead`): **Envio de mensagem**, **Mensagem respondida**, **Sem resposta**, **Ligação realizada**, **E-mail enviado**, **Reunião agendada**, **Outro**. Qualquer valor fora da lista é rejeitado com `400`.
- `POST /dashboard/comercial/leads/{id}/contatos` cria o registro: valida tipo, define `DataContato` (default `DateTime.Now`, editável), `Observacao` (1000 caracteres), auditoria; **atualiza `LEAD_UltimoContato` do lead com a data do contato**.
- `GET /dashboard/comercial/leads/{id}/contatos` lista os contatos do lead ordenados por `DataContato` decrescente.
- `DELETE /dashboard/comercial/contatos/{id}` remove o contato e **recompõe `UltimoContato` do lead com a maior `DataContato` restante** (se não houver contatos restantes, mantém o valor atual do lead).
- Badge visual "aguardando resposta" (âmbar) no card do Kanban e no modal quando o **último tipo de contato for "Envio de mensagem"**.
- Esse badge **não** é a flag "cliente viu o contato": o primeiro é derivado do tipo do último contato, a segunda é marcada manualmente e não se sincroniza com o histórico.
- **Etapas do funil continuam movidas apenas pelo drag** (drag & drop); o registro de contato **não move o lead** automaticamente entre etapas.
- Exclusão em cascata configurada no DbContext: apagar o lead remove todos os seus contatos.

