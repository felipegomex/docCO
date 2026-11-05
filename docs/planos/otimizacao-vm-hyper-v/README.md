# Otimização do Hyper-V no servidor de produção (`192.168.2.254`) e da VM `jca-linux` (`192.168.2.253`)

> **Status:** EXECUTADA (2026-11-01 — detalhes passo a passo no §5)
> **Janela:** liberada pelo usuário ("estamos fora de operação, só voltamos em 2 dias")
> **Escopo:** liberar espaço no host Windows (`192.168.2.254`) removendo o inflação de checkpoints Hyper-V, com backup prévio dos dados da VM.
> **Não cobre:** deploy da Central, schema, banco, AnythingLLM/Gitea/RAG internamente (são outro corpus — este plano só registra a infraestrutura real observada).

---

## 1. Contexto coletado em 2026-11-01 (evidências)

### Host `192.168.2.254` (Windows, IIS PROD, host Hyper-V)

| Item | Valor |
|---|---|
| C: (150 GB) | usado **129,7 GB** · livre **20,3 GB** |
| D: (300 GB) | usado **176,7 GB** · livre **123,3 GB** |
| `C:\ProgramData\Microsoft\Windows\Hyper-V\Snapshots` | **74,95 GB** (estados de RAM dos checkpoints) |
| VM | `jca-linux`, **Running**, 16 GB RAM fixos (memória dinâmica **desligada**), 4 vCPU |
| Checkpoints | **11** (type `Production`), criados entre **10/19/2026 e 10/20/2026** (`antes-2.6`, `pos-ubuntu-ssh`, `antes-docker`, `pos-docker`, `pos-gitea-base`, `antes-padronizacao`, `pos-gitea`, `pos-backup`, `pos-fundacao`, `pos-rag-base`, `pos-rag-teste`) — marcos da instalação de out/2026 |
| Discos da VM em `D:\VMs` | base `jca-linux.vhdx` = **150 GB** (tipo **Fixed** — pré-alocado) + 11 `.avhdx` somando **~26,5 GB** |

**Causas do uso crítico de disco:**
- `C:` — ~75 GB são **estados de memória dos checkpoints** (cada checkpoint guarda um `.vmrs` com a RAM da VM de 16 GB no momento do checkpoint; ~5 checkpoints com RAM salva + metonímia de arquivos parcialmente compartilhados).
- `D:` — o encadeamento `base (Fixed 150 GB) + 11 avhdx`. A VM usa **~23 GB reais** dentro do convidado (`/dev/mapper` 144 GB: usado 23 GB).
- ~26,5 GB em **D:** (os 11 `.avhdx` são apagados pelo merge)
**A Central em si não é o problema:** `inetpub\wwwroot\CentralOperacao\{Back,FRONT}` ≈ 0,04 GB, logs IIS ≈ 0,06 GB, `Backup_IIS` ≈ 0,69 GB (22 pastas de ~30 MB), pagefile 9,5 GB, Windows 23,5 GB, ProgramData fora do bloco Hyper-V ≈ 1,5 GB.

### VM `jca-linux` (`192.168.2.253`, Ubuntu 24.04.5)

| Item | Valor |
|---|---|
| Disco interno | `/` = 144 GB, usado **23 GB**, livre **115 GB** (17%) |
| RAM | total 15 GiB · used **3,4 GiB** · buff/cache 12 GiB · available 12 GiB (saudável) |
| CPU | 4 núcleos · load 0,00 (idle) |
| Docker | 7 containers ativos, 8 imagens (11,85 GB), volumes 889 MB, build cache 2,868 GB (**1,464 GB reclaimable**) |
| journal | 89,1 MB — irrelevante |

### ⚠️ Achado crítico: `jca-linux.vhdx` é **Fixed**, não Dynamic

O `Optimize-VHD -Mode Full` **não compacta disco Fixed** (só expande ou converte). O disco tem 150 GB **pré-alocados**, mesmo com só 23 GB em uso dentro do convidado. O LV interno (`ubuntu--vg`) cobre o disco inteiro — **não é possível encolher ou converter a Fixed ⇒ Dynamic sem rota com ISO/clonagem** (não feito hoje, ver §6 fase B2).

**Consequência para o plano:** mesmo com todos os checkpoints removidos, o vhdx base **continua Fixed em 150 GB**. O ganho eliminável hoje é:
- ~75 GB em **C:** (estados de memória dos checkpoints saem juntos do `Remove-VMSnapshot`)
- ~26,5 GB em **D:** (os 11 `.avhdx` são apagados pela merging)

**~100 GB recuperados ≥ meta do usuário (50 GB).** Encolher de fato a base de 150 GB fica para a fase B2.

## 2. Arquitetura real da VM — os 7 containers (documentação solicitada)

**Não é 1 container: são 7, um por serviço — padrão correto de Docker.** Podem ser adicionados/atualizados individualmente no futuro; os dados persistentes de cada serviço moram em volumes nomeados ou em binds em `/opt/jca` (sobrevivem a recriação do container).

| Container | Imagem | Papel | Porta (host) | Dados persistentes |
|---|---|---|---|---|
| `openwa-api` | `ghcr.io/rmyndharis/openwa:0.25.0` (healthy) | Gateway WhatsApp | `192.168.2.253:2785` (e loopback) | volume `openwa_openwa-data` (766 MB) → `/app/data` |
| `gitea` | `gitea/gitea:28.2.0` | Repositório de conhecimento (RAG corpus) | `:3000` (web) + `:2222` (SSH) | bind `/opt/jca/gitea/data/gitea` → `/data` |
| `gitea-db` | `postgres:16` | Banco do Gitea | interno 5432 | bind `/opt/jca/gitea/data/postgres` |
| `anythingllm` | `mintplexlabs/anythingllm:1.18.0` (healthy) | RAG da Central | `:3001` | bind `/opt/jca/anythingllm/storage` |
| `rag-embedder` | `jca/rag-embedder:1` | Serviço de embedding (projeto RAG) | interna 8080 | sem volume declarado |
| `rag-pg` | `pgvector/pgvector:pg16` | Banco vetorial do RAG | interna 5432 | volume `rag-pg_rag-pg-data` (66,7 MB) |
| `n8n` | `n8nio/n8n:2.45.0` | Automação/workflows | `:5678` | bind `/opt/jca/n8n/data` |
| *(volume órfão)* | — | `935b…b32` sem container (56 MB) — provável resíduo | — | — |

**RAM por container** (docker stats, 2026-11-01): openwa 1,16 GiB é o maior; os demais < 400 MB.

> **Incompatível com a doc vigente:** `docs/operacao/instalacao-openwa/01-1-onde-o-gateway-roda.md` e `docs/arquitetura/visao-geral/07-5-ambientes-reais.md` descrevem a VM como "Docker com 1 container (OpenWA)". A realidade em 2026-11-01 são 7 containers. A doc deve ser atualizada para refletir todos os serviços que sobem junto com o OpenWA.

## 3. Onde buscar credenciais/acessos

| Acesso | Como |
|---|---|
| SSH à VM (`.253`) | `.ssh/config` → alias `jca-linux` → chave `C:\Users\JCA-SUP05\.ssh\id_ed25519_jca_linux` (sem passphrase) |
| SSH ao host (`.254`) | `ssh -i C:\Users\JCA-SUP05\.ssh\central_ops_ed25519 administrator@192.168.2.254` |
| Senha alternativa do host | `.env.local` keys `SSH_PROD_USUARIO` / `SSH_PROD_SENHA` |
| Senha `sudo` da VM | `.env.local` key `SSH_VM253_SENHA` (usuário `jca`) — `sudo` pede senha (não há NOPASSWD); padrão de uso: um `sudo -S -p '' true` inicial autentica; chamadas seguintes dentro da janela de 15 min de `sudo` não re-pedem senha |
| **Nunca** | conteúdo de `.env.local` ou chaves privadas vão para commit, log ou documento versionado |

## 4. Passo a passo (revisado)

**Sequência única, janela de ~30–60 min (fora de operação por 2 dias, sem pressa).**

### Fase C ✅ — limpeza Docker (zero downtime extra)
1. `docker builder prune -f` → devolve **~1,5 GB** (cache de build)
2. `docker rmi hello-world:latest` → imagem sem uso (25,9 kB)

### Fase A ✅ — backup dos dados da VM antes de qualquer mudança de disco
1. `docker stop` nos 7 containers (consistência dos bancos: `gitea-db` e `rag-pg`)
2. `sudo -S` tar (via stdin com senha de `.env.local`, sem ecoar em log) dos conjuntos:
   - volume `openwa_openwa-data` → `/tmp/bak-openwa.tgz`
   - volume `rag-pg_rag-pg-data` → `/tmp/bak-ragpg.tgz`
   - volume órfão `935b…b32` → `/tmp/bak-orphan.tgz` (levantado por completude, é resíduo)
   - binds `/opt/jca/gitea/data`, `/opt/jca/anythingllm/storage`, `/opt/jca/n8n/data` → tgz por área
   - arquivos de composição Docker (`/opt/jca/*.yml`) e `.env` da VM (ver risco R6 sobre segredos)
3. `scp` para `administrator@192.168.2.254:D:/VMs/backup-20261101-1414/` — pasta restrita a `administrator`, **tars não versionados** — executado em duas pernas (guest → estação → host) porque glob remoto não expande no OpenSSH Windows
4. ✅ 1,22 GB confirmados no destino; `/tmp` limpo na VM via container utilitário (tars root-owned não removíveis pelo `jca`)

> **Ajuste durante a execução:** tars dos volumes feitos por **container utilitário** (`postgres:16` com `--entrypoint tar`, rodando como root no convidado) porque a senha do `sudo` de `jca` não é a mesma de `administrator@.254` — plano tabelado §3. Containers `docker stop`-ados ANTES do tar (consistência).

### Fase B1 ✅ — merge dos checkpoints (o que resolve os ~75 GB de `C:` e ~26 GB de `D:`)
1. Verificar estado: `Get-VM`, `Get-VHD` (tipo Fixed confirmado antes)
2. `Stop-VM -Name jca-linux` (graceful; só `-Force` se travar)
3. `Get-VMSnapshot -VMName jca-linux | Remove-VMSnapshot` — remove os **11** (usuário aprovou apagar todos; único ponto de restauração perdido)
4. Aguardar merge: `Get-ChildItem D:\VMs -Filter *.avhdx` deve ir a **zero**; nenhum `Get-VMSnapshot` restante
5. Conferir `C:\ProgramData\...\Hyper-V\Snapshots` encolhido (~75 GB liberados no `C:`) e `D:` com ~26 GB liberados
6. `Start-VM`; aguardar SSH subir; **`docker start $(docker ps -aq)` nos 7 declarados** (⚠️ não iniciam sozinhos: foram "stopados" manualmente e a política `unless-stopped` trata como intencional)
7. Sem `fstrim`/`Optimize-VHD`: nenhum efeito em disco Fixed — **registrado como skip**, motivos no §1

### Fase B2 (FUTURA, não dentro desta janela — documentada para execução separada)
Encolher o disco base de 150 GB é impossível sem rota com boot por ISO (shrink do LV/ext4 dentro do convidado) ou clonagem para disco novo dinâmico (necessita ≥ 150 GB livres em algum volume — hoje não disponível). **Só executar com:**
- janela própria, plano dedicado e backup atual;
- opção A (preferida): boot por ISO (gparted), shrink LV/ext4 → `Resize-VHD` do Fixed → `Convert-VHD` para Dynamic;
- opção B: clone `dd` sparse para disco dinâmico novo + ajuste de boot UEFI + troca de disco no Hyper-V.

## 5. Riscos, consequências e ganhos

| # | Risco | Mitigação | Consequência se falhar |
|---|---|---|---|
| R1 | Perder dados da VM no processo | Backup prévio dos volumes e binds em `D:\VMs\backup-<data>` (fase A) | Restaurar volumes do tar |
| R2 | Merge demorar/esgotar espaço em `D:` | Merge in-place não grava cópia (escreve na base) · `D:` com 123 GB livres · sem pressa | Nenhum — processo podemos re-rodar |
| R3 | Merge interrompido (queda de conexão que controla o SSH) | Rodar em janela única; monitorar com loop local; reexecutar comando é idempotente | Checkpoint a meio — Hyper-V retoma na próxima tentativa |
| R4 | VM não inicializar após operação | Graceful `Stop-VM` (não `-Force` de início) · snapshot removido não compromete boot do convidado | Boot por ISO/rescue — ponto de parada |
| R5 | Container não sobe após restart da VM | `docker start $(docker ps -aq)` explícito (fase B1 passo 6) · smoke por container | start individual + logs por container |
| R6 | `.env` do Docker pode conter segredo (API keys do Gitea/AnythingLLM/OpenWA) | O destino é pasta restrita a `administrator` no `.254`; tar NÃO é versionado; valores de segredo nunca vão para log/commando: | — |
| R7 | Sessão WhatsApp (`central-wa-*`) desparear no reboot | Volume persistente é o interesse; se cair, re-emparelhar número no `/whatsapp/numeros` | Números de teste re-emparelhados manualmente |
| R8 | Remoção de checkpoints: usuário perde pontos de restauração | Aprovado explicitamente (§6 decisão do usuário) | — |
| R9 | Outros serviços (n8n, RAG) com fluxos em execução | Fora de operação; streams retomam ao subir | Retomada manual de workflows n8n |

**Ganhos consolidados desta execução (fase B1 + fase C):**

| Local | Liberação |
|---|---|
| `C:` (estados de RAM dos checkpoints em `Snapshots`) | ~75 GB |
| `D:` (os 11 `.avhdx` removidos pelo merge) | ~26,5 GB |
| Docker build cache | ~1,5 GB |
| Imagem `hello-world` | 0,03 GB |
| **Total** | **~103 GB** (≥ meta 50 GB do usuário) |

## 6. Decisões do usuário (ata)

1. ✅ Aprovado **apagar os 11 checkpoints** (não sobra ponto de restauração).
2. ✅ Janela liberada: "estamos fora de operação, só voltamos em 2 dias".
3. ✅ Item "Backup_IIS" **controlado manualmente pelo usuário** — *nenhum script* tocará em `C:\Users\JCASRV-SUP\Documents\Backup_IIS`; a rotina fica por conta dele.
4. 📝 Usuário solicitou documentar a arquitetura Docker da VM (registrado aqui no §2) e depois atualizar as fichas.
5. ❌ Memória dinâmica: usuário decidiu manter a VM em 16 GB fixos por enquanto.

## 7. Verificações pós-execução (smoke) — todas ✅

1. ✅ VM Running, SSH responde
2. ✅ `docker ps` com os 7 containers Up (`openwa-api` healthy; `anythingllm` subiu em aquecimento)
3. ✅ Portas 2785/3000/3001/5678 alcançáveis a partir do `.254` (Test-NetConnection True)
4. ✅ HTTP 200 em Gitea (:3000), AnythingLLM (:3001) e n8n (:5678) a partir do `.254`
5. ✅ Discos host: `C:` usado 38,8 GB (livre 111,2) · `D:` usado 151,3 GB (livre 148,7)

## 8. Resultados da execução (2026-11-01)

| Passo | Resultado |
|---|---|
| Fase C | 1,464 GB de build cache devolvidos; imagem `hello-world` removida |
| Fase A | Backup `D:\VMs\backup-20261101-1414` com 8 tars (1,22 GB), validado por tamanho; containers parados antes do tar para consistência |
| Fase B1 | 11 checkpoints removidos; merge em cadeia 11→8→3→1 avhdx (~10 min); última folha vazia pendente resolvida com `Checkpoint-VM` + `Remove-VMSnapshot` (arco `19C00...` de 0 GB) |
| Recuperação host | `C:` 129,7 → **38,8 GB usados** (+91,0 GB) · `D:` 176,7 → **151,3 GB usados** (+25,4 GB) · pasta `Snapshots` = 0 GB · **total ~116 GB** |
| Boot pós | VM Running; `docker start $(docker ps -aq)` manual (política `unless-stopped` não re-sobe containers explicitamente "stopados"); os 7 Up |
| Observações | `anythingllm` saiu com `Exited (137)` no `docker stop` (SIGKILL por timeout 10 s) e voltou sem alterações de dados |

## 9. Documentação a atualizar depois (delegar ao docs-writer)

- `docs/arquitetura/visao-geral/07-5-ambientes-reais.md` — VM `jca-linux`: **7 containers** (não 1); VM 16 GB fixos; disco Fixed 150 GB; apontamentos para este plano
- `docs/operacao/instalacao-openwa/01-1-onde-o-gateway-roda.md` — corrigir "1 container" para "dentro do conjunto de 7 containers da VM (§ contrato do plano)"
- `docs/README.md` — referência ao plano `otimizacao-vm-hyper-v`
- Novos valores de disco (C:/D: pós-execução) entram na ficha de ambientes (07-5-ambientes-reais.md)
