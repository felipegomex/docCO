---
id: 04-3-controllers-e-endpoints.03
titulo: "3.3 Agenda"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 610
ficha: docs/modulos/agenda.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Agenda](../../../modulos/agenda.md).

### 3.3 Agenda

Todos os endpoints usam JWT. Leituras usam a política `leitura`; mutações usam `validacao`.

| Método e caminho | Corpo/parâmetros | Resultado principal |
|---|---|---|
| `GET /api/v1/agenda/eventos` | `inicio`, `fim`, `responsavelId?`, `funcaoId?` | Eventos no intervalo, convertidos para Brasília |
| `GET /api/v1/agenda/eventos/{id}` | Sem corpo | Detalhe e participantes |
| `POST /api/v1/agenda/eventos` | `AgendaCriarRequest` | Cria evento após regras de tipo e conflito do responsável |
| `POST /api/v1/agenda/eventos/lote` | `AgendaCriarLoteRequest` | Cria recorrência diária, semanal ou mensal após regras de tipo e conflito do responsável |
| `PUT /api/v1/agenda/eventos/{id}` | `AgendaAtualizarRequest` | Atualiza responsável ou evento conforme permissão; participantes e SLA podem ser substituídos |
| `DELETE /api/v1/agenda/eventos/{id}` | Sem corpo | Exclui quando permitido |
| `PATCH /api/v1/agenda/eventos/{id}/mover` | `AgendaMoverRequest` | Move e verifica conflito do responsável |
| `GET /api/v1/agenda/tipos` | Sem corpo | Tipos ativos |
| `GET /api/v1/agenda/funcoes` | Sem corpo | Funções ligadas a funcionários legados ativos |
| `GET /api/v1/agenda/operadores` | Sem corpo | Operadores ativos |

Os contratos estão em [`AgendaDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Implantacao/AgendaDtos.cs).

`inicio`/`fim` de `GET /eventos` são normalizados por `TimeZoneHelper.BrasiliaParaUtc` (`Services/Implantacao/TimeZoneHelper.cs`): sem fuso = horário de Brasília; `Kind=Utc` (string ISO com `Z` enviada pela Home) passa intacto; `Kind=Local` respeita o fuso da máquina. O helper antigo lançava `ArgumentException` com `Kind=Utc` → **500** no endpoint e agenda vazia na tela (corrigido em 2026-10-07; teste `TimeZoneHelperTests`). O ctor estático resolve `America/Sao_Paulo` (IANA) com fallback para `E. South America Standard Time` no Windows. Todos os `catch (Exception)` do `AgendaController` registram `_logger.LogError` e devolvem `detalhe = ex.Message` em Development; `CriarEventosLote` devolve `201` explícito sem `CreatedAtAction` quando a resposta vem sem id.

