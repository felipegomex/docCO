---
id: regras-negocio.08
titulo: "Sincronização tarefa–agenda"
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
tokens_estimados: 324
ficha: docs/modulos/agenda.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Agenda](../../modulos/agenda.md).

## Sincronização tarefa–agenda

**Situação: parcial; a sincronização existe, mas é unilateral e heurística.**

- A sincronização só é chamada em `MudarColunaAsync`.
- Nomes de coluna que contenham `REUNIAO`/`REUNIÃO` ou `TREINAMENTO`/`CAPACITACAO` geram evento de reunião/treinamento; `MARCO`, `ENTREGA` ou `MILESTONE` geram evento do tipo `Outro`.
- O serviço procura um evento existente pelo mesmo `ProjetoId` e por `Titulo.Contains(tarefa.Titulo)`. Se não encontrar, cria um; se encontrar, atualiza título, descrição, datas, tipo e visibilidade.
- A data usada é `Tarefa.DataPrevisao ?? DateTime.Now`; `DataEntrega` não é a fonte dessa sincronização.
- O responsável do novo evento é `ResponsavelId ?? CriadorId`; não há validação de conflito, participante, idempotência ou vínculo direto por ID de tarefa.
- Não foi localizada sincronização inversa agenda–tarefa, nem sincronização na criação/edição direta da tarefa. Portanto a regra não é bidirecional nem completa.

