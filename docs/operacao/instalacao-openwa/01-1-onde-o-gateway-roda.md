---
id: instalacao-openwa.01
titulo: "1. Onde o gateway roda"
tipo: operacao
area: operacao
sistema: central
tags: [openwa, gateway, whatsapp, docker]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 1100
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 1. Onde o gateway roda

| Item | Valor | Origem da informação |
|---|---|---|
| Hospedagem | VM `192.168.2.253` — nome Hyper-V `jca-linux`, **Ubuntu 24.04.5 LTS**, 16 GB RAM fixos (memória dinâmica desligada), 4 CPU, disco interno com 115 GB livres | verificação na VM em 2026-10-21; RAM fixa e disco re-verificados em 2026-11-01 |
| Onde a VM vive | Hyper-V do `192.168.2.254` (o **servidor IIS de produção**) | documentação única do módulo em [`README.md`](../../../material-de-projeto/atendimento-whatsapp/README.md) |
| Outros serviços da VM | o gateway **não** é o único container do host: em 2026-11-01 a VM roda **7 containers Docker** — além do OpenWA (`openwa-api`), `gitea` (`:3000` web + `:2222` SSH) e `gitea-db` (PostgreSQL do Gitea, interno), `anythingllm` (`:3001`, RAG da Central), `rag-embedder` (embedding do projeto RAG, interna 8080), `rag-pg` (pgvector, banco vetorial, interno) e `n8n` (`:5678`, automação) | verificação na VM em 2026-11-01; inventário completo em [`planos/otimizacao-vm-hyper-v/README.md`](../../planos/otimizacao-vm-hyper-v/README.md) |
| Servidores da Central | `192.168.2.130` (homologação) e `192.168.2.254` (produção) — IIS `CentralOperacao-Back` na `1009` e `CentralOperacao-Front` na `1010` | [documentação única do módulo](../../../material-de-projeto/atendimento-whatsapp/README.md) e [`deploy.md`](../deploy.md) |
| Modo de execução | **Docker** — imagem oficial fixada `ghcr.io/rmyndharis/openwa:0.25.0` (o projeto **não** é compilado do fonte) | execução de 2026-10-21 |
| Diretório do compose | `~/openwa` (clone do projeto na tag `v0.25.0`) + `docker-compose.override.yml` próprio | execução de 2026-10-21 |
| Recursos do container | `shm_size: 2gb`, `mem_limit: 4g`, `pids_limit: 2048` | parâmetros efetivos em 2026-10-21 (§3.2) |
| Porta do gateway | `2785` — API em `/api`, dashboard em `/` | README do OpenWA v0.25.0 |
| Alcance da porta | liberada **apenas** para `192.168.2.130` e `192.168.2.254`; responde também em `127.0.0.1:2785` para a administração local | verificação em 2026-10-21 (`.254` → 200; `.5` → bloqueado) |
| Banco do gateway | **SQLite local** + armazenamento local (perfil mínimo); sem Redis, sem fila, sem PostgreSQL e sem MinIO | `.env` da VM |
| Motor de sessão | `whatsapp-web.js`, no máximo **2 sessões** simultâneas | `.env` da VM (`ENGINE_TYPE`, `MAX_CONCURRENT_SESSIONS`) |
| Estado | volume nomeado `openwa_openwa-data`; o container sobe sozinho com o daemon do Docker (`restart: unless-stopped`) — ⚠️ a política trata container explicitamente parado com `docker stop` como intencional: ele **não** volta após reboot do host/VM; se isso ocorrer, use `docker start $(docker ps -aq)` | execução de 2026-10-21; comportamento de reinício re-verificado em 2026-11-01 (fase B1 do plano) |
| Sessões | **nenhuma** desde o reset de 2026-10-27 (`GET /api/sessions` = `[]`); o piloto usou `central-wa-7` — `ready`, telefone `558592644863` ("Felipe Gomes Corporativo"), registrada em `tbwhatsapp_numero` | verificação em 2026-10-27 (reset) e em 2026-10-21 (piloto) |

**Por que a VM, e não o `.254`.** A instalação nativa no `192.168.2.254` foi **descartada**: o `.254` é o **servidor IIS de produção** da Central, e o gateway é uma peça separada, com ciclo de vida e risco próprios. O piloto parcial no `.254` (OpenWA, tarefas agendadas, logs e scripts temporários) foi **removido**. Na VM o caminho **oficial** do OpenWA fica disponível (Docker, com dashboard e imagem oficial), o gateway não disputa recursos com o IIS e o firewall entre os hosts é explícito: `2785` sai da VM só para os dois servidores da Central, e `1009` é liberado em cada servidor da Central só para a VM.

### 1.1 O que o gateway **não** é

- **Não** é o banco do módulo: o histórico operacional (atendimentos, mensagens, eventos, fila de webhook) fica no **SQL Server `192.168.2.154`**; o SQLite do gateway é interno dele.
- **Não** faz parte do deploy do IIS: `deploy.ps1` publica a Central e **não** publica nem migra o gateway.
- **Não** é acessível pelo navegador: o frontend nunca fala com o OpenWA — tudo passa pela API da Central (`/api/v1/whatsapp`).

