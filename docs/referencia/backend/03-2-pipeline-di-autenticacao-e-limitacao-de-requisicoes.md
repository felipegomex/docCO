---
id: sec.03-2-pipeline-di-autenticacao-e-limitacao-de-requisicoes
titulo: "2. Pipeline, DI, autenticação e limitação de requisições"
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
tokens_estimados: 1819
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/backend.md](../backend.md).

# 2. Pipeline, DI, autenticação e limitação de requisições

### 2.1 Pipeline HTTP

Fora de Development, o tratador global de exceções e HSTS são habilitados antes. A ordem comum do pipeline em [`Program.cs`](../../../backend/Central_BackEnd/Program.cs) é:

1. `UseRouting()`;
2. `UseRateLimiter()`;
3. `UseCors("Angular")`;
4. Swagger, quando habilitado;
5. `UseAuthentication()`;
6. `UseAuthorization()`;
7. `MapControllers()`;
8. `MapHub<WhatsAppHub>("/hubs/whatsapp")`.

O tratador global de exceções devolve 500 genérico. A política CORS `Angular` permite cabeçalhos e métodos configurados, credenciais e somente as origens cadastradas no código. Não há `MapFallbackToFile` nem chamada automática a `Database.Migrate()` na raiz de composição. O hub SignalR do WhatsApp é **canal de notificação de saída**, não uma API REST: a fonte da verdade continua sendo a API versionada, e o cliente que recebe um evento relê o dado autenticado (§4.5.3).

### 2.2 DbContext

`AppDbContext` é scoped. Em Development, `Database:UseSqlServer` seleciona SQL Server ou InMemory; nos demais ambientes o provider é SQL Server. A design-time factory usa SQL Server local para permitir geração de migrações.

**Nível de compatibilidade declarado no provider.** Nos dois ramos SQL Server (Development com `Database:UseSqlServer = true` e ambiente publicado), o registro usa `sql => sql.UseCompatibilityLevel(NivelCompatibilidadeSqlServer)`, com a constante `NivelCompatibilidadeSqlServer = 100` em [`Program.cs`](../../../backend/Central_BackEnd/Program.cs). O nível é o real de `dbCENTRAL_OPERACAO`, `dbBUSINESS_HML` e `dbBUSINESS`. Sem informá-lo, o EF Core 8 traduz `Contains` de lista em `OPENJSON(... '$')` (nível 130+); com ele, volta a gerar `IN (...)`. O caso que expôs a falta foi o SLA do WhatsApp: a consulta falhava e o `try/catch` de `CalcularSlaAsync` engolia o erro, então o SLA simplesmente não aparecia.

### 2.3 Dependências de autenticação

| Configuração | Aplicação |
|---|---|
| Chave JWT | Variável de ambiente tem precedência sobre `Jwt:Key` |
| Issuer/audience/lifetime | Configuração `Jwt` |
| Algoritmo | HMAC SHA-256 |
| Validação | Issuer, audience, expiração e assinatura; clock skew de um minuto |
| Credencial de acesso | Bearer token; o refresh token não é Bearer |

### 2.4 Lifetimes do DI

| Lifetime | Registrations |
|---|---|
| Scoped | `AppDbContext`, `IAuthService`, `IConhecimentoService`, `IConhecimentoSyncService`, `GiteaApiClient`, `IPasswordValidationService` |
| Scoped | `IPermissaoService` → `PermissaoService`, `IOperadorAdminService` → `OperadorAdminService` e `IAuthorizationHandler` → `PermissaoHandler` |
| Singleton | `IAuthorizationPolicyProvider` → `PermissaoPolicyProvider` e `IAuthorizationMiddlewareResultHandler` → `AuditoriaNegacaoHandler` |
| Scoped | Todos os serviços de Implantação: tipos, colunas, projetos, cards, jornada, tarefas, dashboards, agenda e auditoria |
| Scoped | Serviços do Database Explorer, exceto a conexão |
| Scoped | Serviços do Dashboard Integrado: comercial, administrativo e financeiro |
| Controller | **Chat**: `ChatSsoController` faz o SSO passthrough para o AnythingLLM (`GET /api/chat/sso`); o chat em si é servido pelo AnythingLLM em iframe e não tem serviços de conversa registrados (`ConhecimentoServiceCollectionExtensions.AddConhecimento` mantém apenas configuração do sistema, Gitea e Conhecimento) |
| Scoped | **Módulo de atendimento WhatsApp**: `IAtendimentoService`, `IWhatsAppNumeroService`, `IWhatsAppMetricasService`, `IWhatsAppWebhookService`, `IWhatsAppParametroService`, `IWhatsAppRespostaRapidaService`, `IWhatsAppConfiguracaoService`, `IWhatsAppSlaService`, `IWhatsAppRetencaoService`, `IContatoUnificacaoService`, **`IContatoVinculoService`** (vínculo do contato com o ERP — Fase 01 do novo ciclo), **`IEtiquetaService`** (catálogo de etiquetas — Fase 06), **`IContatoAdminService`** (leitura administrativa de contatos — Fase 08), **`IWhatsAppFotoContatoService`** (foto do contato — Fase 3 do plano), `IWhatsAppEnvioRecuperador` e **`WhatsAppFinalizacaoAutomaticaService`** (finalização automática — Fase 2 do plano) |
| Typed client | `IOpenWaClient` → `OpenWaClient` (`AddHttpClient<IOpenWaClient, OpenWaClient>`): base address e timeout (30 s) vêm de `WhatsApp:BaseUrl` (com a **barra final normalizada** por `NormalizarBaseUrl`); `BaseUrl` vazio **não derruba o boot** |
| Scoped/Singleton | **Tempo real do WhatsApp (Fase 09)**: `IWhatsAppRealtimeService` → `WhatsAppRealtimeService` (**singleton**, usa `IHubContext<WhatsAppHub>`), `IWhatsAppTypingHandler` → `WhatsAppTypingHandler` (**scoped** — depende de `IWhatsAppHubAuthorization` e `IWhatsAppTypingExternoService`, ambos scoped e sobre `AppDbContext`; como singleton, a dependência cativa impedia o backend de instanciar o hub em Development), `IWhatsAppTypingExternoService` → `WhatsAppTypingExternoService` (scoped) e `IWhatsAppHubAuthorization` → `WhatsAppHubAuthorization` (scoped); `AddSignalR()` registra a infraestrutura do hub. O debounce de 3 s continua valendo porque o estado (`Ultimos`) é `static` |
| Hosted services | `WhatsAppWebhookProcessor` (drena a fila durável de webhooks), `WhatsAppEnvioRecuperadorJob` (reenvio de mensagens presas), `WhatsAppRetencaoJob` (limpeza de mídia agendada), `WhatsAppArquivamentoJob` (arquivamento diário de Finalizados) e `WhatsAppFinalizacaoAutomaticaJob` (finalização de Resolvidos, uma volta por minuto — Fase 2 do plano) |
| Singleton | `BruteForceGuard`, `IAcessoTicketService`, `IDatabaseConnectionService`, `ICifrador`, `AuditoriaFila` + `IAuditoriaFila` |
| Infraestrutura | `IHttpClientFactory` por `AddHttpClient()` |
| Estático | `SegurancaHelper`, `TimeZoneHelper`, `RegrasPrazo`, `PerfilOperador` e extensão de atraso |

`IPasswordValidationService` não guarda prova de senha: o serviço é `Scoped` e o estado que dura é o **ticket** emitido pelo `AcessoTicketService`, que é `Singleton` (registro na tabela acima junto com `BruteForceGuard`) e por isso se perde quando o pool do IIS recicla.

### 2.5 Limitação de requisições

| Política | Chave | Limite |
|---|---|---|
| `login` | IP remoto | 5/min fora de Development; 50/min em Development |
| `validacao` | Política: usuário se `User` já estiver autenticado; caso contrário, IP | 50/min fora de Development; 100/min em Development |
| `leitura` | Política: usuário se `User` já estiver autenticado; caso contrário, IP | 100/min |
| `chat-conhecimento` | Claim autenticado ou IP | 12/min fora de Development; 30/min em Development |
| `conhecimento` | Claim autenticado ou IP | 60/min fora de Development; 120/min em Development |

Como `UseRateLimiter()` vem antes de `UseAuthentication()` no pipeline atual, a política não encontra o principal preenchido por esse middleware e usa o IP. Não há fila; a rejeição usa HTTP 429.

