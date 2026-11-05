---
id: raiz.09
titulo: "GestoresController (`/gestores`)"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 694
ficha: docs/modulos/ideias.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Ideias, Melhorias e Erros](../../../modulos/ideias.md).

## GestoresController (`/gestores`)

Fonte: [GestoresController.cs](../../../../backend/Central_BackEnd/Controllers/GestoresController.cs). **6 operações** no código (2026-10-18). Todo acesso exige autenticação.

| Método | Rota | Quem pode | Observação |
|---|---|---|---|
| `GET` | `/gestores` | autenticado + **Administrador** | lista gestores; 403 para os demais |
| `GET` | `/gestores/{id}` | — | Operação presente no código sem linha no catálogo anterior — descrição a completar direto do código. |
| `POST` | `/gestores` | autenticado + **Administrador** | corpo: `operadorId` + `funcaoId` (null = gestor geral); usuário vem do token. **Regra 2026-10-13**: no máximo um gestor ativo por função — se a função já tem gestor, o anterior é **removido** e o novo assume, com `substituiuOperadorId/Nome` na resposta. Cadastrar o mesmo operador na mesma função é idempotente. 400 com motivo (operador inativo inexistente, função inexistente, dois admins gravando ao mesmo tempo) |
| `DELETE` | `/gestores/{id}` | autenticado + **Administrador** | **204** removido, **404** inexistente, **400** quando é o último gestor geral ativo. Substituiu o antigo `PATCH /{id}/ato`: remover gestor passou a ser exclusão, não inativação |
| `GET` | `/gestores/resumo` | — | Operação presente no código sem linha no catálogo anterior — descrição a completar direto do código. |
| `GET` | `/gestores/minhas-funcoes` | autenticado | ajuda de UI (`{ funcaoIds }` — funções que o logado gerencia; Kanban e Agenda usam para exibir ações) — não é concessão; a escrita revalida no serviço |

- [Limitação] `tboperador.FUNCAO_ID` está parcialmente sem backfill: a função do criador — que define qual gestor administra a solicitação — pode estar vazia em registros antigos. Ver [`FuncionarioLegado.cs`](../../../../backend/Central_BackEnd/Models/Implantacao/FuncionarioLegado.cs).
- [Limitação] `GestorService.SalvarAsync` **não** exige que o gestor pertença à função que administra: qualquer operador ativo pode ser gestor de qualquer função.
- [Limitação] A regra "um gestor ativo por função" é garantida no serviço (com mensagem) e pelo índice `UX_tbgestor_Funcao` — criado por [`GestorUnicoPorFuncao_Idempotente.sql`](../../../../backend/Central_BackEnd/Migrations/Sql/GestorUnicoPorFuncao_Idempotente.sql), que também resolve dados que já violavam a regra. Sem o script aplicado, a checagem de aplicação continua valendo, mas sem a garantia do banco em caso de corrida.

