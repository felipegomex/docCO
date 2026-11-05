---
titulo: "Chat e Conhecimento — como funciona e o que falta"
status: "vivo"
revisado_em: "2026-10-30"
fonte: [backend/Central_BackEnd/Controllers/ConhecimentoController.cs, backend/Central_BackEnd/Controllers/ChatSsoController.cs, backend/Central_BackEnd/Services/Configuracao/ConfiguracaoSistemaService.cs, backend/Central_BackEnd/Services/Configuracao/ChaveMestraProvider.cs, backend/Central_BackEnd/Services/Conhecimento/GiteaTokenService.cs, backend/Central_BackEnd/Services/Conhecimento/ConhecimentoPathPolicy.cs, scripts/knowledge-sync/sync.py]
testes: [backend/Central_BackEnd.Tests/ConhecimentoPathPolicyTests.cs, backend/Central_BackEnd.Tests/ConfiguracaoSistemaTests.cs, backend/Central_BackEnd.Tests/GiteaTokenServiceTests.cs, backend/Central_BackEnd.Tests/ConhecimentoIntegracoesControllerTests.cs, scripts/knowledge-sync/test_sync.py]
---

# Chat e Conhecimento — como funciona e o que falta

> **Atualização (2026-11-03):** o chat passou a ser servido pelo AnythingLLM em
> iframe, com SSO pela Central (`GET /api/chat/sso`, commit `cb36ad3`). As seções
> sobre agentes/RAG próprio descrevem a fase anterior; o Conhecimento segue como
> documentado.

Este documento substitui os planos extensos `00-PLANO-EXECUCAO.md` e
`01-PLANO-GERAL.md` (arquivados em `_arquivo/`). Ele descreve o funcionamento
real, o estado de cada fase e a lista acionável do que resta. Para detalhe de
módulo, use [`docs/modulos/chat-conhecimento.md`](../../modulos/chat-conhecimento.md);
para o diagnóstico operacional, [`docs/checklist-hml-chat.md`](../../checklist-hml-chat.md).

## 1. Como funciona hoje

```
Gitea jca/jca-knowledge@main  (fonte de verdade dos documentos)
        │  leitura via API REST (token de leitura)
        ▼
VM Ubuntu 192.168.2.253 · /opt/jca/sync
   sync.py ── compara SHA-256 ──► manifesto /opt/jca/sync/state/manifest.json
        │  upload/remoção
        ▼
AnythingLLM em 192.168.2.253:3001 (1 workspace por função do agente)
        │  POST /api/v1/workspace/{slug}/vector-search
        ▼
Central .NET 8 · backend :1009 · front :1010
   POST /api/chat/perguntar  →  fontes (documento, caminho, trecho, score)
```

Regras estruturais:

- **O Gitea é a única fonte.** Documento não entra em workspace sem estar no
  repositório, em pasta mapeada e fora das pastas excluídas.
- **Quem decide o destino é a pasta.** Não existe workspace por documento: o
  documento cai em todos os workspaces que mapeiam a pasta onde ele está.
- **O sync é incremental e idempotente.** Só sobe o que mudou (SHA-256), remove
  o que saiu e não duplica em execuções repetidas.
- **Nada de escrita manual no AnythingLLM.** Workspace criado à mão fica fora do
  manifesto e é sobrescrito/órfão na próxima execução.

## 2. Peças e onde vivem

| Peça | Onde | Função |
|---|---|---|
| Repositório de conteúdo | `http://192.168.2.253:3000/jca/jca-knowledge` | documentos, `_config/workspaces.yml`, `MATRIZ-SETOR-WORKSPACE.md` |
| Sync | VM `192.168.2.253`, `/opt/jca/sync/app/sync.py` | lê o Gitea, mantém manifesto e workspaces |
| Manifesto | `/opt/jca/sync/state/manifest.json` | hash + workspaces + localização de cada documento |
| Status do sync | `/opt/jca/sync/state/sync-status.json` | `pendente`, `em_andamento`, `concluida`, `erro` |
| Trigger | VM `:8090`, `POST /api/sync`, header `X-Central-Sync-Key` | agenda o sync (202) e não empilha execução (409) |
| Agendamento | cron diário + `jca-sync-trigger.service` | sync periódico e sob demanda |
| AnythingLLM | `http://192.168.2.253:3001` | workspaces, embeddings, `vector-search` |
| API de conhecimento | `ConhecimentoController` | árvore, documento, diff, histórico, aprovar, rejeitar, sync, OAuth |
| API de chat | `ChatSsoController` | `sso` (iframe do AnythingLLM; agentes/perguntar/provedor removidos em `cb36ad3`) |
| API de integrações | `ConhecimentoController` (`admin/integracoes`) | status do token do Gitea e da cifra; grava o token do Gitea |
| Política de caminhos | `ConhecimentoPathPolicy` | valida staging, destinos e conteúdo |
| Segredos no banco | `tbconfiguracao_sistema` + `ConfiguracaoSistemaService` | chave-mestra (`acessos.chave_cifra`, cifrada pelo DataProtection) e token do Gitea (`gitea.service_token`, AES-GCM) |
| Chaveiro do DataProtection | `CENTRAL_DATAPROTECTION_PATH` → `%ProgramData%\Central\DataProtection-Keys` → `App_Data/DataProtection-Keys` | persiste a chave que decifra a chave-mestra; **perder esse diretório torna a chave-mestra ilegível** |
| SQL/migration | `Migrations/Sql/Chat*_Idempotente.sql` | `tbchat_agente`, `tbchat_auditoria`, `tbchat_workspace_funcao`, `tbconhecimento_*`, `tbchat_provedor`, `tbconfiguracao_sistema` |

## 3. Mapa pasta → workspace

Definido em `_config/workspaces.yml`, lido pelo sync a cada execução (o mapa
embutido no código é só fallback).

| Workspace | Pastas |
|---|---|
| `suporte` | `04-operacoes/`, `02-produtos/`, `03-integracoes/`, `05-infraestrutura/` |
| `implantacao` | `04-operacoes/`, `02-produtos/`, `03-integracoes/` |
| `desenvolvimento` | `04-operacoes/`, `02-produtos/`, `03-integracoes/` |
| `ciaa` | `04-operacoes/`, `02-produtos/`, `03-integracoes/` |
| `financeiro` | `09-financeiro/`, `10-comercial/` |
| `glossario` | `06-glossario/` |
| excluído | `07-revisao/`, `08-pendente/`, `BAIXA_QUALIDADE/`, `99-historico/`, `_templates/`, `_config/` |

Existem no AnythingLLM, mas **não** são alimentados pelo mapa: `comercial` e
`teste-embedding`.

Destinos aceitos na aprovação (`ConhecimentoPathPolicy.RootsDestino`):
`01-empresa/`, `02-produtos/`, `03-integracoes/`, `04-operacoes/`,
`05-infraestrutura/`, `06-glossario/`, `09-financeiro/`, `10-comercial/`.
Staging (nunca destino): `07-revisao/` (efetivo) e `08-pendente/revisar/` (legado).

## 4. Ciclo de vida de um documento

1. **Revisão** — documento nasce em `07-revisao/` (staging). Não é indexado.
2. **Aprovação** — revisor escolhe a `pasta_destino`; o backend move o arquivo
   da staging para a pasta final em uma branch e abre PR (ou commita direto,
   conforme configuração do repositório).
3. **Merge** — o commit chega em `main`.
4. **Sync** — disparado após a aprovação, pelo cron ou manualmente; o `sync.py`
   lê o Gitea, e cada documento elegível é enviado aos workspaces da sua pasta.
5. **Chat** — `POST /api/chat/perguntar` faz `vector-search` no workspace da
   função do operador e devolve a resposta com as fontes citadas.

## 5. Caminho A — aprovação pela tela (implementado, não publicado)

1. Operador abre `/conhecimento/nova` e o documento vai para `07-revisao/`.
2. Revisor abre o documento, escolhe `pasta_destino` e clica **Aprovar**
   (`POST /api/conhecimento/documento/aprovar`).
3. Backend usa o token de serviço do Gitea para criar branch, mover o arquivo e
   abrir PR; o token é **configurado pela tela admin**
   (`/conhecimento/configuracao`) e fica **cifrado no banco**
   (`tbconfiguracao_sistema`, chave `gitea.service_token`).
4. Sync agendado; o chat passa a citar o documento após a indexação.

Falhas esperadas: **400** destino fora de `RootsDestino` (inclusive staging),
**503** quando **não há token de serviço em lugar nenhum** — nem no banco nem em
`Gitea:ServiceToken` na configuração.

## 6. Caminho B — promoção manual (o que funciona hoje)

Sem deploy, com acesso ao Gitea:

1. Mover 1 documento revisado de `07-revisao/` para a pasta final via branch +
   PR (push direto em `main` é bloqueado pela proteção).
2. Mergear (squash ou rebase).
3. Disparar o sync (VM ou Central) e conferir `sync-status.json`.
4. Perguntar no chat e verificar a fonte; repetir o sync e confirmar que o
   manifesto não muda.

Exemplo já executado (2026-10-30): `04 - Actyon Cob - Batimento de Carteira.md`
movido para `04-operacoes/` no commit `87ab8bd`.

## 7. Endpoints

| Rota | Auth | Observação |
|---|---|---|
| `POST /api/chat/perguntar` | operador | pergunta + fontes; provedor resolvido em runtime |
| `GET /api/chat/agentes` | operador | `base_status` por agente |
| `GET/PUT /api/chat/provedor`, `POST /api/chat/provedor/testar` | admin | chave cifrada, `503` sem `Acessos:ChaveCifra` |
| `GET /api/conhecimento/{permissao,arvore,documento,diff,historico,sincronizacao}` | operador | leitura |
| `PUT/POST /api/conhecimento/documento*`, `aprovar`, `rejeitar` | revisor | escrita no Gitea |
| `POST /api/conhecimento/sincronizar` | revisor | chama o trigger da VM |
| `GET/POST /api/conhecimento/gitea/oauth/*` | operador | conexão da conta |
| `GET /api/conhecimento/admin/integracoes` | admin | status mascarado do token e da cifra; nunca devolve o token |
| `PUT /api/conhecimento/admin/integracoes/gitea` | admin | grava o token do Gitea cifrado; `204`, `400` inválido ou `503` sem cifra |

## 8. Estado por fase

| Fase | Estado | Evidência |
|---|---|---|
| 0 — provar o agente | concluída | `vector-search` no `glossario` com 6 documentos |
| 1 — provedor pela tela | código pronto, **não publicado** | `/api/chat/provedor` responde 404 em `.130` e `.254` |
| 2 — revisão e pasta destino | código pronto, **não publicado** | `PastaDestino` no `aprovar`; testes de política verdes |
| 3 — mapa volátil e sync único | concluída | `_config/workspaces.yml` (PR #5, `fd8b9b8`), trigger e cron |
| 4 — árvore nova no Gitea | concluída | PR #6, squash `088611c`: 147 renames, `.gitkeep` em `09-financeiro/` e `10-comercial/` |
| 5 B — promoção manual | concluída | `87ab8bd`, sync `concluida`, manifesto 7 docs, `vector-search` sem duplicar |
| 5 A — aprovação pela tela | **bloqueada** | depende de deploy + token de serviço |
| 6 — observabilidade | parcial | status por agente e logs existem; `scripts/chat-diagnostico.ps1` **não existe** |
| Segredos no banco (chave-mestra e token do Gitea) | SQL aplicado nos dois bancos, **sem commit/deploy** | `tbconfiguracao_sistema`, `20261030170000_ConfiguracaoSistema` e endpoints `admin/integracoes`; 64/64 verdes em Chat/Conhecimento |

Estado do corpus em `main` (2026-10-30): 221 `.md` no total; `07-revisao/` com
146 (147 migrados, 1 promovido); `08-pendente/revisar/` só `.gitkeep`;
`BAIXA_QUALIDADE/` com 53. Manifesto: 7 documentos — `glossario` 6, `suporte`,
`implantacao`, `desenvolvimento` e `ciaa` com 1 (o mesmo).

## 9. O que falta (na ordem)

1. **Revogar o PAT exposto no chat** e gerar outro, exclusivo do servidor.
2. **SQL:** aplicado nos dois bancos em 2026-10-30 (4 SQLs/4 migrations:
   `ChatConhecimento`, `ChatAgente`, `ChatProvedor` e `ChatConfiguracaoSistema`);
   reexecutar `scripts\aplicar-chat-sql.ps1 -ConfirmarBanco <banco>` é idempotente
   e só necessário se o schema for recriado.
3. **Comprovar o 503** em HML: aprovar sem token de serviço em lugar nenhum
   (nem banco nem `Gitea__ServiceToken`).
4. **Salvar o token do Gitea pela tela admin** (depois de publicar, item 6) —
   não é mais obrigatório definir `Acessos__ChaveCifra` nem `Gitea__ServiceToken`
   por variável de ambiente: a chave-mestra é gerada na primeira subida e gravada
   em `tbconfiguracao_sistema` (cifrada pelo DataProtection), e o token é gravado
   cifrado pela tela `/conhecimento/configuracao`. As variáveis continuam válidas
   como **override** (precedência da chave-mestra do appsettings/env).
5. **Fazer backup do diretório do chaveiro do DataProtection** antes de publicar:
   `CENTRAL_DATAPROTECTION_PATH` → `%ProgramData%\Central\DataProtection-Keys` →
   `App_Data/DataProtection-Keys` (fallback).
6. **Publicar backend e front** em `.130` e `.254` (`deploy.ps1` com `$true`
   como argumentos; `preflight-schema.ps1` aborta se o schema faltar).
7. **Rodar smoke** nos dois (`scripts\smoke.ps1 -Servidor ...`): login fictício
   deve dar 401.
8. **Conectar a conta no Gitea** pela UI (`gitea_conectado=true`).
9. **Aceite da Fase 5 A**: `/conhecimento/nova` → escolher pasta → Aprovar →
   commit + sync → chat cita a fonte. Negativos: 400 destino inválido, 503 sem token.
10. **Aceite da Fase 5 B pelo chat**: `POST /api/chat/perguntar` com "Como fazer
    batimento de carteira no Actyon Cob?" deve citar
    `04-operacoes/04 - Actyon Cob - Batimento de Carteira.md`.
11. **Decidir a revisão por par**: `main` exige 1 aprovação e o Gitea bloqueia
    auto-approve; os dois merges usaram `force_merge` (admin).
12. **Criar ou remover do plano** `scripts/chat-diagnostico.ps1` (Fase 6).

## 10. Runbook

VM (`jca@192.168.2.253`), sempre carregando o ambiente — sem isso o sync falha
com `GITEA_API_URL ausente`:

```bash
cd /opt/jca/sync && set -a && . ./.env && set +a
python3 app/sync.py --dry-run            # não escreve nada
cat state/sync-status.json               # status da última execução
tail -20 trigger.log                     # chamadas do trigger
```

Trigger local (o segredo vem do `.env`, nunca da linha de comando):

```bash
curl -s -X POST -H "X-Central-Sync-Key: $KNOWLEDGE_SYNC_SECRET" \
  http://127.0.0.1:8090/api/sync
```

Servidores:

```powershell
.\deploy.ps1 -BuildFrontend $true -BuildBackend $true -Publicar 0
.\deploy.ps1 -BuildFrontend $false -BuildBackend $false -Publicar 1 -Backup 1
.\scripts\smoke.ps1 -Servidor 192.168.2.130   # HML
.\scripts\smoke.ps1 -Servidor 192.168.2.254   # PROD
```

Gates locais:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
powershell -ExecutionPolicy Bypass -File .\scripts\check-docs.ps1 -ExigirFrontmatterSecoes
python -m unittest discover -s scripts\knowledge-sync -p "test_sync.py" -v
```

## 11. Critérios de aceite

- `07-revisao/` nunca aparece no manifesto (0 documentos) nem no log do sync.
- Contagens do manifesto: `glossario` 6; demais workspaces só com o que for
  aprovado; reexecução do sync não altera essas contagens.
- `POST /api/chat/perguntar` cita caminho e trecho do documento correto.
- Aprovar com destino de staging ou inválido responde **400**; sem
  `Gitea:ServiceToken` responde **503**.
- `validate.ps1`, `check-docs.ps1 -ExigirFrontmatterSecoes` e `test_sync.py`
  verdes.

## 12. Riscos e decisões abertas

| Risco | Impacto | Mitigação |
|---|---|---|
| `main` protegida sem revisor efetivo (`force_merge`) | revisão por par decorativa | definir segundo revisor ou aceitar explicitamente o override de admin |
| Sync lento (~3 min por documento em 4 workspaces) | fila do trigger ocupada | disparar 1 por vez; o trigger devolve 409 |
| Teste de escrita manual no AnythingLLM | documento órfão fora do manifesto | nunca criar workspace/documento à mão |
| Perda do diretório do chaveiro do DataProtection | a chave-mestra gravada em `tbconfiguracao_sistema` fica **ilegível** (o código não sobrescreve chave ilegível) e a cifra/token param de funcionar | backup do diretório do chaveiro (`%ProgramData%\Central\DataProtection-Keys` ou `CENTRAL_DATAPROTECTION_PATH`) a cada deploy e antes de trocar de servidor |
| Ficha `chat-conhecimento.md` acima de 4k tokens | aviso no `check-docs` | enxugar quando o módulo estabilizar |
| `scripts/chat-diagnostico.ps1` documentado e inexistente | diagnóstico da Fase 6 não reproduzível | criar o script ou remover a referência |

## 13. Histórico

- `fd8b9b8` — mapa volátil de workspaces (PR #5).
- `088611c` — árvore da Fase 4: `07-revisao/`, `09-financeiro/`, `10-comercial/` (PR #6).
- `87ab8bd` — promoção de 1 documento para `04-operacoes/` (PR #7, rebase, autor `jca`).
- 2026-10-30 — sync oficial na VM `concluida`; manifesto 7 documentos; reexecução
  sem duplicação.

## 14. Regras que não mudam

- Sem deploy, SQL ou migration sem pedido explícito.
- Sem segredo em diff, log, PR ou mensagem.
- O valor do token do Gitea **nunca** volta ao cliente: a API devolve só
  `gitea_token_mascara` (`••••abcd`) e a origem (`banco`/`configuracao`).
- Sync oficial só na VM, com `MANIFEST_PATH=/opt/jca/sync/state/manifest.json`.
- Documento só entra no RAG por pasta mapeada e aprovada.
- Em divergência entre este documento e o código, parar e perguntar.
