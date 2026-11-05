---
id: componentes.06
titulo: "5. Versionamento e superfície HTTP"
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
tokens_estimados: 1718
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 5. Versionamento e superfície HTTP

### Versionamento

O backend registra API Versioning com as seguintes regras:

- a versão padrão é `1.0`;
- a URL pública usada pelo frontend é `/api/v1`;
- a versão também pode ser informada pelo header `X-Api-Version`;
- `AssumeDefaultVersionWhenUnspecified` permite que uma requisição sem versão seja interpretada como `1.0` quando a rota aplicável atende a esse padrão.

As APIs sem versão confirmadas são o Chat (`GET /api/chat/sso`, iframe do AnythingLLM) e o Conhecimento (`/api/conhecimento/*`). O contrato anterior `/api/rag-proxy` não é mais ativo.

- A versão de release do frontend é gravada no build (`scripts/deploy/deploy.ps1`: `Version=1.0.0` no `BUILD_INFO.txt` e `version.json` com `{ "versao", "commit", "dataUtc" }` em `frontend/dist/central-conhecimento-actyon/browser/version.json`) e consumida pelo frontend via `BuildInfoService` (`GET /version.json`, fallback `APP_VERSION`). O Guia do Sistema (`/faq-sistema`) exibe a badge de versão (SemVer, commit e data de deploy) e o mini sistema de navegação por módulo.

### Rotas por módulo

| Módulo | Prefixo canônico | Autorização geral |
|---|---|---|
| Autenticação | `/api/v1/auth` | Login e refresh sem `[Authorize]`; logout e me autenticados |
| RBAC | `/api/v1/permissoes`, `/api/v1/operadores`, `/api/v1/cargos` | `GET /permissoes/minhas` e leituras de operadores/cargos exigem JWT; escritas de operador usam policy `perm:aba.sistema.operadores:<acao>`; `GET /permissoes/recursos` é exclusivo do `GESTOR_MASTER` |
| Acessos Empresas | `/api/v1/acessos` (31 operações) | Operador autenticado, sem policy por papel |
| Agenda | `/api/v1/agenda` | Operador autenticado |
| Database Explorer | `/api/v1/database` | Operador autenticado |
| Implantação | `/api/v1/implantacao/projetos`, `/api/v1/implantacao/tarefas`, `/api/v1/implantacao/colunas-kanban`, `/api/v1/implantacao/tipos-projeto` | Leitura autenticada; escrita de tipos e do **catálogo** de colunas restrita a `Administrador` (a **visão do setor** é do gestor da função ou do administrador); projetos e tarefas usam as regras de negócio do módulo |
| Dashboard | `/api/v1/implantacao/dashboard` | Operador autenticado |
| Dashboard administrativo | `/api/v1/admin/dashboard` | Role `Administrador` |
| Dashboard Integrado | `/api/v1/dashboard/comercial`, `/api/v1/dashboard/administrativo`, `/api/v1/dashboard/financeiro` | Comercial e administrativo com operador autenticado; financeiro com role `Administrador` |
| Chat | `/api/chat/sso` (iframe AnythingLLM) | JWT; sem API REST de agentes; workspace vem da config do AnythingLLM |
| Conhecimento | `/api/conhecimento/*` | JWT; revisor por função ou administrador, com exceção do callback OAuth anônimo |
| Tempo real WhatsApp (SignalR) | `/hubs/whatsapp` (**não é API REST**) | `[Authorize(Policy = "perm:aba.suporte.whatsapp:Ler")]`; JWT por `access_token` na query **apenas** nesse path. Os métodos `EntrarNoAtendimento`/`EntrarNaFila`/`EntrarNoOperador`/`Digitando` são notificação de saída; a leitura/escrita continua na API |

### `/api/v1/auth/*`

| Método e rota | Corpo | Autorização | Comportamento |
|---|---|---|---|
| `POST /api/v1/auth/login` | JSON `{ "usuario": string, "senha": string, "lembrarAcesso": boolean }`; `lembrarAcesso` pode ser omitido e assume `false` quando ausente | Pública | Valida o operador, cria access token e refresh token, grava `cc_refresh` e devolve `accessToken`, `expiraEmSegundos` e `user`; o JSON inclui `refreshToken: ""` |
| `POST /api/v1/auth/refresh` | Sem corpo obrigatório; o frontend envia `{}` | Pública, mas exige o cookie `cc_refresh` | Revoga o token recebido, cria access e refresh tokens, substitui o cookie e devolve o mesmo formato de sessão; o JSON inclui `refreshToken: ""` |
| `POST /api/v1/auth/logout` | Sem corpo obrigatório | Bearer válido | Revoga o refresh token atual, apaga os cookies e retorna `204` |
| `GET /api/v1/auth/me` | Sem corpo | Bearer válido | Devolve o usuário oficial associado ao token |

Em login e refresh, o JSON transporta `refreshToken: ""`; o valor bruto permanece somente no cookie HttpOnly. O cookie de lembrança `cc_lembrar` guarda apenas a preferência de persistência; ele não contém o refresh token.

### `/api/v1/acessos/*` — Acessos Empresas

Rota versionada, `[Authorize]` **sem policy por papel** e **31 operações**. Os fluxos e contratos estão detalhados em [`05-ENDPOINTS.md`](../../referencia/endpoints.md); aqui vale a forma:

| Método e rota | Corpo ou query | Autorização | Comportamento |
|---|---|---|---|
| `POST /api/v1/acessos/clientes/{id}/autorizar` | JSON `{ "usuario": string, "senha": string, "finalidade": "VISUALIZAR" \| "EDITAR", "lembrar": boolean }` | Bearer válido | **Única porta de entrada do módulo**: confere o `usuario` digitado contra o JWT (divergência = 401 auditada, sem queimar tentativa) e a senha do operador, e devolve o ticket (`ticket`, `expiraEm`, `finalidade`, `usoUnico`, `validoPor`) que as demais operações exigem no header `X-Acesso-Ticket`. TTL de `VISUALIZAR`: 120 s, ou 300 s com `lembrar: true`. `id = 0` emite ticket de criação |
| `GET /api/v1/acessos/estado-cifra` | Sem corpo | Bearer válido | `implementacao`, `ehCifradoDeVerdade`, `eventosAuditoriaPerdidos` — diagnóstico da proteção e da fila de auditoria |
| `GET /api/v1/acessos/clientes` | Query `grupoId`, `busca`, `somenteAtivos`, `page`, `pageSize` | Bearer válido | Lista **paginada** (`Paginado<T>`, padrão 20/máx. 200) com as flags derivadas. **Nunca devolve credencial** |
| `GET /api/v1/acessos/{rdp,vpn}/{id}/senha` | Sem corpo | Bearer válido | **Os únicos 2 endpoints que devolvem senha.** Cada chamada enfileira `REVEAL_SENHA` antes de responder |

A senha não é guardada em nenhum cache. Quem guarda a prova de senha é o ticket de acesso (`AcessoTicketService`): `VISUALIZAR` dura 2 minutos (5 quando o modal marca "Lembrar") e é reutilizável na mesma empresa; `EDITAR` dura 2 minutos e é de **uso único**, consumido na primeira operação aceita. O ticket é escopado por `ClienteAcessoId` e por `OperadorId`, então a senha de uma empresa não abre outra. `PasswordValidationService` apenas compara a senha enviada no gate (com o `usuario` conferido contra o JWT) e delega o bloqueio por tentativas ao `BruteForceGuard`. As demais operações são a leitura de grupos (legada `tbcliente_grupo`), o CRUD de unidades (incluindo a criação a partir do ERP e o **POST agregado com acessos**) e dos 5 tipos de acesso, com soft delete e auditoria.

### Cifra das credenciais

`ICifrador` está registrado no DI com `CifradorAesGcm` (AES-256-GCM, envelope `v1:iv:tag:ciphertext`, chave em `Acessos:ChaveCifra`): as senhas novas são gravadas cifradas. `CifradorEmBranco` permanece no código apenas como implementação de teste/fallback e **não está registrada**. `Decifrar` aceita valores sem o prefixo `v1:` para ler linhas antigas, mas **isso não comprova que o backfill terminou** — a cobertura precisa ser contada. Detalhes e procedimento em [`banco-dados.md`](../banco-dados.md).

