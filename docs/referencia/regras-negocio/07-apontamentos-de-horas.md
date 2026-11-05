---
id: regras-negocio.07
titulo: "Apontamentos de horas"
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
tokens_estimados: 416
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Apontamentos de horas

**Situação: parcial.**

- Cada apontamento pertence a uma tarefa, tem operador, data, horas, observação e usuário de inclusão. `Horas` é `decimal(5,2)` e deve ser maior que zero.
- A soma dos apontamentos é mantida em `Tarefa.HorasRealizadas`; o valor persistido é inteiro e usa `Math.Ceiling` da soma. A criação, alteração e exclusão recalculam esse campo.
- A data vazia na criação assume `DateTime.Today`. Se `OperadorId` vier preenchido, o serviço usa esse valor; caso contrário, usa o operador autenticado.
- Alteração e exclusão verificam se o apontamento pertence ao operador autenticado, salvo quando o controller informa `ehAdmin`.
- O caminho administrativo de apontamentos **funciona**: o controller calcula `ehAdmin` com `User.IsInRole("Administrador")`, claim `eh_admin = "true"` ou claim `perfil_id = "A"`, e o token emitido pelo [AuthService.cs](../../../backend/Central_BackEnd/Services/AuthService.cs) carrega os três (o `perfil_id` já normalizado). O administrador edita e exclui apontamento de outro operador; nos demais casos, a regra é só "próprio apontamento".
- A criação aceita um `OperadorId` enviado na requisição sem validar se ele existe ou está ativo. A atualização e a exclusão não registram uma nova linha em `tbauditoriaimplantacao`; apenas a criação chama o auditor de implantação.

