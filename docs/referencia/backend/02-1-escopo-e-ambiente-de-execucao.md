---
id: sec.02-1-escopo-e-ambiente-de-execucao
titulo: "1. Escopo e ambiente de execução"
tipo: referencia
area: backend
sistema: central
tags: [backend, aspnet, controllers, services]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 621
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/backend.md](../backend.md).

# 1. Escopo e ambiente de execução

### 1.1 Fontes principais

| Fonte | Papel factual |
|---|---|
| [`backend/Central_BackEnd/Central_BackEnd.csproj`](../../../backend/Central_BackEnd/Central_BackEnd.csproj) | .NET 8, nullable, EF Core SQL Server/InMemory e JWT |
| [`backend/Central_BackEnd/Program.cs`](../../../backend/Central_BackEnd/Program.cs) | Composition root, DI, autenticação, middleware, seeds e pipeline |
| [`backend/Central_BackEnd/Data/AppDbContext.cs`](../../../backend/Central_BackEnd/Data/AppDbContext.cs) | DbSets, chaves, índices e relacionamentos EF |
| [`backend/Central_BackEnd/Controllers`](../../../backend/Central_BackEnd/Controllers) | Controllers versionados e exceções Chat/Conhecimento |
| [`backend/Central_BackEnd/Services`](../../../backend/Central_BackEnd/Services) | Serviços de domínio, segurança, integração e Database Explorer |
| [`backend/Central_BackEnd/Models`](../../../backend/Central_BackEnd/Models) | Entidades EF, espelhos legados e contratos sem persistência |
| [`backend/Central_BackEnd/Dtos`](../../../backend/Central_BackEnd/Dtos) | Requests e responses versionados |
| [`backend/Central_BackEnd/Migrations`](../../../backend/Central_BackEnd/Migrations) | Histórico EF e snapshot atual |

### 1.2 Stack

| Item | Estado no código |
|---|---|
| Runtime | ASP.NET Core 8 / C# com nullable habilitado |
| Persistência principal | EF Core 8 com SQL Server; os dois ramos SQL Server declaram `UseCompatibilityLevel(100)` |
| Desenvolvimento | Provider InMemory quando `Database:UseSqlServer` é falso |
| Versionamento de API | Segmento de URL ou cabeçalho; versão padrão 1.0 |
| Autenticação | JWT Bearer HMAC SHA-256 |
| Integrações | AnythingLLM (recuperação de fontes), provedor de resposta selecionado por `Chat:Provedor` (`claude`/`gemini`/`openai-compatible`, via `IChatRespostaProvedor`) e Gitea no Chat/Conhecimento |
| Banco explorador | `Microsoft.Data.SqlClient` em conexão separada |
| OpenAPI | Swagger condicional por configuração |
| Testes | `backend/Central_BackEnd.Tests` (xunit + FluentAssertions, `net8.0`); contagem por suíte em 5.5 |

