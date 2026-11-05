---
id: 06-5-features-e-status-de-uso.02
titulo: "Comportamentos que não devem ser "simplificados""
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 472
ficha: docs/modulos/README.md
---
> Parte de [5. Features e status de uso](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### Comportamentos que não devem ser "simplificados"

1. **Gate de senha no detalhe.** `DetalheClienteComponent` **não tem gate próprio**: quem libera a entrada é a listagem, que paga a senha e emite o ticket de `VISUALIZAR` (2 min, ou 5 min com "Lembrar" — reusável na mesma empresa) — por isso o `ngOnInit` só chama `GET /estado-cifra` e `GET /clientes/{id}`, e a troca de abas não reprompta. Já as ações sensíveis (novo, editar, inativar, revelar) chamam `SenhaGateService.exigir(clienteId, 'EDITAR')`, que abre o modal e emite ticket de **uso único** a cada vez. O gate não substitui `authGuard` — é uma segunda barreira de uso, e o backend cobra o ticket de qualquer jeito (`403` sem ele). O modal pede **usuário digitado + senha**; o backend confere o usuário contra o JWT (401 auditado sem queimar tentativa).
2. **Auto-esconder em 30 s.** `EXPIRACAO_SENHA_S = 30`: toda senha revelada some sozinha depois de 30 segundos. Os timers são guardados e limpos no `ngOnDestroy`.
3. **Olho só em RDP e VPN.** `podeRevelar()` retorna verdadeiro apenas nessas abas. AnyDesk, Banco e Actyon Web não têm botão — reflete os endpoints de revelação do backend.
4. **Aviso da cifra no topo.** Lista e detalhe chamam `GET /estado-cifra` e mostram o estado real da proteção enquanto `ehCifradoDeVerdade` for `false`. É informação de segurança, não decoração: não remova sem antes ligar a criptografia.
5. **Toast e padrão do Dashboard.** Reutiliza `DashboardToastService`, `DashboardToastComponent` e `mensagemErro` de `features/dashboard/shared`, como os demais módulos CRUD.

