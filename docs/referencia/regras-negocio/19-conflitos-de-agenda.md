---
id: regras-negocio.19
titulo: "Conflitos de agenda"
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
tokens_estimados: 465
ficha: docs/modulos/agenda.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Agenda](../../modulos/agenda.md).

## Conflitos de agenda

**Situação: implementada no serviço, com lacunas de concorrência e participantes.**

- A sobreposição usa intervalos semiabertos: `inicio < fimExistente` e `fimExistente > inicio`. Eventos que apenas se tocam no limite não conflitam.
- Para dia inteiro sem `DataFim`, o fim efetivo é o início do dia seguinte; para evento pontual sem fim, o fim efetivo é o próprio início.
- A checagem é feita para o `OperadorId` responsável. Ela não é feita para cada participante.
- `ListarEventosAsync` filtra intervalo, responsável e função, mas não filtra `Visibilidade`; `ObterEventoAsync` também não verifica visibilidade nem propriedade. Assim, `AgendaItem.Visibilidade` é persistido, porém eventos privados podem ser listados ou obtidos por qualquer usuário autenticado. A restrição de responsável/admin existe apenas nas mutações de alterar, mover e excluir.
- `Férias` exige retorno em data posterior e é normalizada como dia inteiro; `Treinamento`/`Daily` não permitem dia inteiro e exigem horário; `Reunião`/`Atendimento` também exigem horário e não permitem span de vários dias; `Pessoal`, `Outro` e ausência de tipo ficam livres.
- Criar, atualizar e mover devolvem `409` quando há conflito, com `mensagem`, lista `conflitos` e `code` (por padrão `CONFLICT_HORARIOS`). O lote valida cada ocorrência antes de salvar.
- Não há restrição de banco que impeja duas requisições concorrentes de criarem sobreposição; a validação é de aplicação. O lote também não demonstra transação atômica entre a validação e a gravação.

