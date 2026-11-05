---
id: visao-geral.07
titulo: "5. Ambientes reais"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 1450
ficha: docs/modulos/README.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 5. Ambientes reais

> URLs, IPs, portas e alvos de dados abaixo são defaults operacionais extraídos de código e scripts, não uma prova do estado atual de um servidor. Confirme cada valor no ambiente antes de usá-lo.

| Ambiente | Frontend | Backend | Persistência e observações |
|---|---|---|---|
| Local, Development | `http://localhost:4200` (default a confirmar) | `http://localhost:1009` (default a confirmar); Swagger em `/swagger` | A seleção depende de `Database:UseSqlServer`: `false` usa EF Core InMemory `CentralDev`; `true` usa SQL Server. O valor efetivo local deve ser confirmado. |
| Homologação de dados para desenvolvimento | — | Endpoint local do backend | Código e scripts usam como referência operacional o SQL Server `192.168.2.154` e o banco `dbBUSINESS_HML`; confirmar rota e configuração efetiva. Credenciais não são reproduzidas aqui. |
| IIS/produção | `http://192.168.2.130:1010` (default operacional a confirmar) | `http://192.168.2.130:1009/api/v1` (default operacional a confirmar); Swagger em `/swagger` | Código e scripts referenciam SQL Server `192.168.2.130`, banco `Central_Conhecimento` e autenticação integrada; confirmar bindings e configuração efetiva. O deploy preserva `appsettings*.json` do servidor. |
| Chat/Conhecimento | APIs sem versão `/api/chat/sso` e `/api/conhecimento/*` | AnythingLLM, Claude e Gitea na VM `192.168.2.253` | Publicados em HML (2026-10-28) e PROD; SQL aplicado em 2026-10-30. O chat é iframe do AnythingLLM autenticado por SSO; não há API REST de agentes |
| **Gateway OpenWA (atendimento WhatsApp)** | — | VM Ubuntu `192.168.2.253` (`jca-linux`, Ubuntu 24.04.5 LTS, 16 GB RAM fixos — memória dinâmica desligada, 4 CPU), escutando em `192.168.2.253:2785` (e em `127.0.0.1:2785` para a administração local); entrega eventos ao backend do ambiente por `POST /api/v1/whatsapp/webhook/openwa` | **No ar, com o piloto em operação na homologação** (2026-10-21): OpenWA **v0.25.0** em **Docker** (imagem oficial fixada `ghcr.io/rmyndharis/openwa:0.25.0`, `shm_size: 2gb`, `mem_limit: 4g`, `pids_limit: 2048`, estado no volume `openwa_openwa-data`), perfil mínimo (SQLite local + armazenamento local), sem Redis e sem fila. A porta publicada é restrita por firewall na cadeia `DOCKER-USER` (o `ufw` **não** restringe porta publicada por Docker). A chave da API e o segredo do webhook ficam **só na VM e nos servidores da Central** (`WhatsApp__ApiKey` / `WhatsApp__WebhookSecret`). **Piloto em operação no `.130`**: número pareado (sessão `central-wa-7`, `ready`, telefone `558592644863`) e Kanban recebendo mensagens reais. **A produção (`.254`) foi publicada em 2026-10-26.** Runbook em [`operacao/instalacao-openwa.md`](../../operacao/instalacao-openwa/INDEX.md). |
| **VM `jca-linux` — infraestrutura Docker (compartilhada)** | — | — | A mesma VM hospeda **7 containers Docker** (uso efetivo em 2026-11-01): `openwa-api` (`ghcr.io/rmyndharis/openwa:0.25.0` — gateway WhatsApp, `:2785`) · `gitea` (`gitea/gitea:28.2.0` — repositório de conhecimento, `:3000` web + `:2222` SSH) · `gitea-db` (`postgres:16` — banco do Gitea, interno 5432) · `anythingllm` (`mintplexlabs/anythingllm:1.18.0` — RAG da Central, `:3001`) · `rag-embedder` (`jca/rag-embedder:1` — embedding do projeto RAG, interna 8080) · `rag-pg` (`pgvector/pgvector:pg16` — banco vetorial do RAG, interno 5432) · `n8n` (`n8nio/n8n:2.45.0` — automação, `:5678`). **Checkpointing zerado em 2026-11-01**: os 11 checkpoints Hyper-V foram removidos (merge completo), `C:` de 129,7 → 38,8 GB usados e `D:` de 176,7 → 151,3 GB no host `192.168.2.254` (~116 GB recuperados). O disco base em `D:\VMs\jca-linux.vhdx` é **Fixed de 150 GB pré-alocados** — encolhê-lo exige rota com ISO/clonagem, registrada como fase B2 futura no plano. Contêineres com política `unless-stopped`: os explicitamente parados com `docker stop` não sobem após reboot do host — usar `docker start $(docker ps -aq)`. Execução, riscos e números completos: [`planos/otimizacao-vm-hyper-v/README.md`](../../planos/otimizacao-vm-hyper-v/README.md). |

> ⚠️ **O banco de homologação é o de produção.** `192.168.2.154 / dbBUSINESS_HML` (SQL Server em `compatibility_level 100`) é o alvo real do backend. **Não existe staging**: toda migration é mudança de schema em produção de primeira vez. Procedimento e riscos em [`09-TROUBLESHOOTING.md`](../../operacao/troubleshooting.md) e [`10-DEPLOY.md`](../../operacao/deploy.md).

O código do frontend define `useMockAuth=false`; a configuração efetiva deve ser conferida no ambiente. A tela de login e a renovação chamam o backend real. O Database Explorer possui precedência própria: variáveis `DB_EXPLORER_*`, depois seção `DatabaseExplorer`, depois configuração vazia; sua disponibilidade deve ser confirmada no servidor.

