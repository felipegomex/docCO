---
id: raiz.07
titulo: "DocsController (`/docs`) — leitura da documentação sincronizada"
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
tokens_estimados: 454
ficha: docs/modulos/README.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

## DocsController (`/docs`) — leitura da documentação sincronizada

Fonte: [DocsController.cs](../../../../backend/Central_BackEnd/Controllers/DocsController.cs) (plano docs-api, fase 2; conferido em 2026-10-19). Rota de classe `api/v{version:apiVersion}/docs`, `[ApiVersion("1.0")]`, `[Authorize]` — como os demais controllers versionados. **Somente leitura**: não há endpoint de escrita; a gravação em `tbdoc_doc` é o MERGE de [`scripts/sync-docs-to-db.ps1`](../../../../scripts/sync-docs-to-db.ps1), fora da superfície da API.

| Método | Rota | Entrada | Resposta |
|---|---|---|---|
| `GET` | `/api/v1/docs` | Sem corpo | `200 List<DocIndiceResumo>` — índice ordenado por `Secao` e `Caminho` com `Caminho`, `Titulo`, `Secao` e `Hash`, **sem `Corpo`** (a listagem nunca puxa o conteúdo integral). |
| `GET` | `/api/v1/docs/{*caminho}` | Path catch-all | `200 DocDetalhe` (`DocIndiceResumo` + `Corpo` markdown e `SincronizadoEm` UTC); `404` para caminho vazio/inexistente ou travessia. |

Notas confirmadas no serviço ([DocumentosService.cs](../../../../backend/Central_BackEnd/Services/Shared/DocumentosService.cs)):

- O caminho pode vir com ou sem o prefixo `docs/` — `NormalizarCaminho` aceita com/sem prefixo, de-duplica barras e **rejeita travessia** (`".."`): o que não normaliza devolve `null` e o controller responde `404`.
- `IDocumentosService.CalcularHash` é estático na interface: 8 primeiros hex em minúsculas do SHA-256 UTF-8 do conteúdo — o mesmo algoritmo do script de sync.

