---
id: raiz.04
titulo: "Agenda"
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
tokens_estimados: 1113
ficha: docs/modulos/agenda.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Agenda](../../../modulos/agenda.md).

## Agenda

Fonte: [AgendaController.cs](../../../../backend/Central_BackEnd/Controllers/AgendaController.cs).

| Método | Rota | Autorização | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|---|
| `GET` | `/api/v1/agenda/eventos` | JWT | `leitura` | Query `inicio`, `fim`, `responsavelId?`, `funcaoId?` | `200 List<AgendaResumo>`. |
| `GET` | `/api/v1/agenda/eventos/{id}` | JWT | `leitura` | Path `id`; sem body | `200 AgendaDetalhe` (inclui `recorrente`, `padraoRecorrencia`, `serieId` e `dataRepeticaoFim` — fim da série = maior `DataInicio` do lote, convertido para horário de Brasília) ou `404`. |
| `POST` | `/api/v1/agenda/eventos` | JWT | `validacao` | Body `AgendaCriarRequest` | `201 AgendaDetalhe`; `400` para regras de data/tipo e `409` para conflito. |
| `POST` | `/api/v1/agenda/eventos/lote` | JWT | `validacao` | Body `AgendaCriarLoteRequest` | `201 AgendaLoteResponse` com `totalCriados` e `eventos`; `400` para intervalo sem dias (`dias.Count == 0` / `eventos.Count == 0`) e demais validações; `409` para conflito. Grava `AGD_SerieId` = Id do 1º evento em todas as ocorrências (2º `SaveChanges`). Resposta sem id devolve `201` explícito (`StatusCode(201)`), nunca `CreatedAtAction` com id nulo. Erro inesperado: `500` logado (`_logger.LogError`), com `detalhe = ex.Message` em Development. |
| `PUT` | `/api/v1/agenda/eventos/{id}` | JWT | `validacao` | Body `AgendaAtualizarRequest` (campos novos `dataRepeticaoFim?`, `padraoRecorrencia?`) | `200 AgendaDetalhe`; `403` para responsável/transferência não autorizada, `404` se ausente e `409` para conflito. Quando `dataRepeticaoFim` vem presente, **reconcilia a série** (`AGD_SerieId`): estreita, alarga e preenche buracos na âncora do menor `DataInicio`, removendo só as ocorrências geradas que ficaram fora (a editada nunca); `dataRepeticaoFim` **vazio/ausente não mexe na estrutura**; `409` é validado só nas datas a criar, **antes** do `SaveChanges`; `dataRepeticaoFim < dataInicio` → `400`. |
| `DELETE` | `/api/v1/agenda/eventos/{id}` | JWT | `validacao` | Path `id`; sem body | `204` ou `404`; `403` para não responsável não administrador. |
| `PATCH` | `/api/v1/agenda/eventos/{id}/mover` | JWT | `validacao` | Body `AgendaMoverRequest` | `200 AgendaDetalhe`; `400` para intervalo, `403` para responsabilidade não autorizada e `409` para conflito. |
| `GET` | `/api/v1/agenda/tipos` | JWT | `leitura` | Sem body | `200 List<TipoEventoResponse>` de tipos ativos. |
| `GET` | `/api/v1/agenda/funcoes` | JWT | `leitura` | Sem body | `200 List<FuncaoResumo>`; filtra funções ligadas a funcionários legados ativos. |
| `GET` | `/api/v1/agenda/operadores` | JWT | `leitura` | Sem body | `200 List<OperadorResumo>` de operadores ativos. |

O conflito é devolvido com `mensagem`, `conflitos` e `code`. O tipo de evento pode impor horário, dia inteiro e regras de recorrência; essas regras estão descritas em [AgendaService.cs](../../../../backend/Central_BackEnd/Services/Implantacao/AgendaService.cs).

**Fuso de `inicio`/`fim` (GET `/eventos`):** data sem fuso (`Kind=Unspecified`) é interpretada como horário de **Brasília**; instante UTC com `Z` (`Kind=Utc`, o formato que o frontend envia via `Date.toISOString()`) é aceito como está; `Kind=Local` respeita o fuso da máquina. A normalização é feita por `TimeZoneHelper.BrasiliaParaUtc` (`Services/Implantacao/TimeZoneHelper.cs`), coberta por `TimeZoneHelperTests` em `backend/Central_BackEnd.Tests`. Antes da correção de 2026-10-07, o formato `Z` lançava `ArgumentException` ("Kind property set correctly") e o endpoint retornava **500** — a Home mostrava a agenda sempre vazia (na época, `/planejamento` e Home eram telas separadas; foram fundidas na `/`); ver [09-TROUBLESHOOTING.md](../../../operacao/troubleshooting.md).

**Limitação de leitura:** `GET /eventos` não filtra `AgendaItem.Visibilidade`, e `GET /eventos/{id}` não verifica visibilidade nem propriedade. A persistência de `Visibilidade` não impede que um usuário autenticado liste/obtenha um evento privado; a checagem de responsável/administrador existe nas mutações. Essa é uma limitação de segurança, não uma regra desejada.

