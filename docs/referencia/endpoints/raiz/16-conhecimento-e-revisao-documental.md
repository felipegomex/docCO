---
id: raiz.16
titulo: "Conhecimento e revisão documental"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - backend/Central_BackEnd/Controllers/ConhecimentoController.cs
  - backend/Central_BackEnd/Services/Conhecimento/GiteaTokenService.cs
testes:
  - backend/Central_BackEnd.Tests/ConhecimentoIntegracoesControllerTests.cs
tokens_estimados: 1250
ficha: docs/modulos/chat-conhecimento.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Chat e Conhecimento](../../../modulos/chat-conhecimento.md).

## Conhecimento e revisão documental

Fonte: [ConhecimentoController.cs](../../../../backend/Central_BackEnd/Controllers/ConhecimentoController.cs) e [ConhecimentoService.cs](../../../../backend/Central_BackEnd/Services/Conhecimento/ConhecimentoService.cs). As rotas não têm `/api/v1`; o controller exige JWT, aplica a política `conhecimento` (60 requisições/minuto em produção; 120 em Development) e revalida permissões no servidor, exceto o callback OAuth, explicitamente anônimo.

| Método | Rota | Acesso | Entrada/parâmetros | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/api/conhecimento/permissao` | JWT | — | `200` com `pode_acessar`, `eh_revisor`, `eh_admin`, `workspaces`, `gitea_conectado`. |
| `GET` | `/api/conhecimento/arvore` | Revisor ou admin | `status?`, `tipo?`, `cliente?` | Árvore/pastas e documentos visíveis no escopo de workspace; `403` sem permissão. |
| `GET` | `/api/conhecimento/documento` | Revisor ou admin | `path` | Conteúdo, frontmatter, SHA, SHA base e último commit. |
| `PUT` | `/api/conhecimento/documento` | Revisor ou admin | `{ path, content, baseSha, mensagem, enviarParaRevisao }` | Salva em branch e PR; `409` se o SHA base divergir. |
| `POST` | `/api/conhecimento/documento/nova` | Revisor ou admin | `{ titulo, tipo, cliente, sistema, modulo, pasta_sugerida, conteudo }` | Cria rascunho em branch/PR sob `08-pendente/revisar/`. |
| `GET` | `/api/conhecimento/diff` | Revisor ou admin | `path`, `baseSha` | Diff entre a revisão e a base atual. |
| `GET` | `/api/conhecimento/historico` | Revisor ou admin | `path` | Até 50 commits e PR associado quando conhecido. |
| `POST` | `/api/conhecimento/documento/aprovar` | Revisor ou admin | `{ path, baseSha, comentario? }` | Aprovação pessoal no Gitea, merge e solicitação de sync; `202` com status/sync. |
| `POST` | `/api/conhecimento/documento/rejeitar` | Revisor ou admin | `{ path, baseSha, motivo }` | Cria PR de rejeição e atualiza o documento com status `rejeitado` no mesmo caminho; o revisor aprova/mescla com OAuth pessoal; `409` em conflito. |
| `POST` | `/api/conhecimento/gitea/oauth/iniciar` | Revisor | — | `200 { url }`; `503` se OAuth não estiver configurado. |
| `GET` | `/api/conhecimento/gitea/oauth/callback` | Anônimo | `code`, `state`, `error?` | Callback OAuth; consome state em cache e redireciona para `/conhecimento`. |
| `GET` | `/api/conhecimento/sincronizacao` | JWT | — | Estado do job: pendente, em andamento, concluída ou erro. |
| `POST` | `/api/conhecimento/sincronizar` | Administrador | — | `202 { status: "agendado" }`; `503` se gatilho não puder ser solicitado. |
| `GET` | `/api/conhecimento/admin/config` | Administrador | — | Funções/workspaces e operadores/vínculos de revisor para a tela administrativa. |
| `PUT` | `/api/conhecimento/admin/workspaces` | Administrador | `{ funcaoId, workspace, ativo }` | Configura o workspace por função; `204`, `400` ou `404`. |
| `PUT` | `/api/conhecimento/admin/revisores` | Administrador | `{ operadorId, funcaoId, ativo }` | Configura vínculo de revisor por função; `204` ou `404`. |
| `GET` | `/api/conhecimento/admin/integracoes` | Administrador | — | `200` com `gitea_token_configurado`, `gitea_token_mascara` (`••••abcd`), `gitea_token_origem` (`banco` \| `configuracao` \| `null`), `cifra_configurada` e `chave_mestra_origem` (`banco` \| `configuracao` \| `null`); **nunca** devolve o token. |
| `PUT` | `/api/conhecimento/admin/integracoes/gitea` | Administrador | `{ token }` | `204` ao gravar o token cifrado (AES-GCM) em `tbconfiguracao_sistema`; `400` se vazio, fora de 16..255 caracteres ou com espaço; `503` sem cifra disponível; `403` para não-admin. |

Os dois endpoints de integrações usam o mesmo `ConhecimentoController` (`[Authorize]` + `EhAdmin`) e rate limit `conhecimento`. O token de serviço é lido do banco (`gitea.service_token`) com fallback para `Gitea:ServiceToken`; a chave-mestra (`acessos.chave_cifra`) é gerada na primeira subida e gravada cifrada pelo DataProtection, com precedência para `Acessos:ChaveCifra` válida no appsettings/env. O cliente recebe apenas máscara e origem.

Em `409` por SHA divergente, a API devolve o SHA e o conteúdo atuais (`sha`, `content` e `mensagem`); a tela mantém o texto local e oferece comparação lado a lado ou recarga da versão atual. Na aprovação, o token de serviço prepara branch/alterações/PR; o OAuth pessoal do revisor aprova e mescla. A rejeição segue o mesmo esquema: token de serviço cria branch/commit/PR com `status: rejeitado` no caminho original e o OAuth pessoal aprova e mescla. Antes de ativar a rejeição, atualize `_config/status-documento.yml` no repositório Gitea para aceitar `rejeitado` como status permitido. Não documentar valores de segredo. O cliente não é fonte de autoridade para função, workspace ou autorização.

A sincronização manual administrativa é `POST /api/conhecimento/sincronizar`; o estado consumido pela Central é `GET /api/conhecimento/sincronizacao`. O serviço chama o gatilho `POST /api/sync` e consulta `GET /api/status` na VM (`/opt/jca/sync/app`, porta **8090**, cabeçalho `X-Central-Sync-Key`); o mapa pasta→workspace vem de `_config/workspaces.yml` no Gitea.

