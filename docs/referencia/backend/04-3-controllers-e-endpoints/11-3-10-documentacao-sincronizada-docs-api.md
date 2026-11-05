---
id: 04-3-controllers-e-endpoints.11
titulo: "3.10 Documentação sincronizada (docs-api)"
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
tokens_estimados: 378
ficha: docs/modulos/chat-conhecimento.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Chat e Conhecimento](../../../modulos/chat-conhecimento.md).

### 3.10 Documentação sincronizada (docs-api)

`DocsController` (`api/v{version}/docs`, `[ApiVersion("1.0")]`, `[Authorize]`) expõe **somente leitura** de `tbdoc_doc` — o espelho da pasta `docs/` carregado pelo script de sync (fase 3 do plano docs-api); não há endpoint de escrita. Contrato das duas rotas (`GET /api/v1/docs` e `GET /api/v1/docs/{*caminho}`, com `404` em caminho vazio/travessia/inexistente) está detalhado em [`endpoints/raiz.md`](../../endpoints/raiz/INDEX.md).

O serviço `IDocumentosService`/`DocumentosService` ([`DocumentosService.cs`](../../../../backend/Central_BackEnd/Services/Shared/DocumentosService.cs), registrado **scoped** em `Program.cs`) usa `AppDbContext` com `AsNoTracking`: `ListarIndiceAsync` devolve o índice por `Secao`+`Caminho` sem corpo (`DocIndiceResumo`) e `ObterPorCaminhoAsync` devolve o documento por demanda (`DocDetalhe`, com corpo e `SincronizadoEm`). A chave de sincronização é `Caminho`; `NormalizarCaminho` (interno) aceita entrada com/sem prefixo `docs/`, de-duplica barras e rejeita travessia `..` (→ `null` → `404`). `IDocumentosService.CalcularHash` é estático na interface — 8 primeiros hex de SHA-256 UTF-8, o mesmo algoritmo do sync.

