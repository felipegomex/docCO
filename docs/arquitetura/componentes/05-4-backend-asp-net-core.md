---
id: componentes.05
titulo: "4. Backend ASP.NET Core"
tipo: arquitetura
area: arquitetura
sistema: central
tags: [arquitetura, componentes, jwt, cors]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 660
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 4. Backend ASP.NET Core

`Program.cs` concentra composição, autenticação, autorização, versionamento, CORS, rate limiting e pipeline HTTP.

- Controllers API são registrados com `AddControllers`.
- EF Core é registrado por `AppDbContext`.
- Em Development, `Database:UseSqlServer` escolhe SQL Server ou InMemory.
- Fora de Development, o backend usa SQL Server.
- JWT Bearer valida issuer, audience, assinatura, algoritmo, lifetime e expiração.
- `UseRateLimiter`, `UseCors`, `UseAuthentication` e `UseAuthorization` precedem `MapControllers`.
- `AddSignalR()` registra a infraestrutura e `MapHub<WhatsAppHub>("/hubs/whatsapp")` expõe o hub do módulo WhatsApp. Ele **não é API REST**: o JWT vai em `access_token` na query **apenas** para esse path (o navegador não envia o header no handshake do WebSocket), e todo o resto da API continua header-only. Os eventos levam só IDs/status/metadados e são publicados **depois do commit**.
- Os controllers validam o token e as roles declaradas, mas não há middleware universal que reconsulte `SeAtivo` a cada requisição; a condição do operador é verificada em login, refresh, `me` e em alguns fluxos de serviço, não uniformemente em todos os endpoints.
- Em ambiente não Development existe handler global de exceção, resposta JSON genérica e HSTS.
- Swagger é condicionado pela flag `SwaggerEnabled`; a variável IIS `ASPNETCORE_SWAGGER_ENABLED` tem outro nome e não corresponde automaticamente à chave lida.

**Regras transversais no backend.** Duas decisões que antes eram tomadas em cada serviço — e, no frontend, em cada tela — vivem agora em classes estáticas de `Services/Shared/`: [`RegrasPrazo`](../../../backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs) (prazo e atraso de tarefa, projeto e etapa) e [`PerfilOperador`](../../../backend/Central_BackEnd/Services/Shared/PerfilOperador.cs) (normalização do `PERFIL_ID` e rótulo do perfil). A primeira decide o que é "atrasado" e **sobe no contrato** (`atrasado` no resumo de projeto, no resumo de tarefa e no card do quadro), para o cliente consumir a decisão em vez de recalcular; a segunda faz o token carregar a letra do perfil **já normalizada**, para que o mesmo operador não seja administrador num fluxo e não administrador em outro. Detalhes em [`07-SERVICES-BACKEND.md`](../../referencia/backend.md).

