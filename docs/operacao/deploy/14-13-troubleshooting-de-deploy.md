---
id: sec.14-13-troubleshooting-de-deploy
titulo: "13. Troubleshooting de deploy"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-30"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 3089
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 13. Troubleshooting de deploy

| Sintoma | Causa provável | Ação |
|---|---|---|
| validate passa, package Release falha | validate usa `dotnet build` Debug | pare no package; não publique |
| working tree sujo no validate | alteração não commitada | commit ou descarte explícito; não empacote assim |
| artefato antigo continua no pacote | `/E` não limpa saídas anteriores | feche processos, limpe apenas diretórios gerados e refaça build/package |
| `BUILD_INFO.txt` ausente | package não foi executado ou foi apagado | rode package completo |
| `BUILD_INFO` diverge do HEAD | commit após package | valide e empacote novamente |
| `-SenhaRemota` ausente no publish | o script exige credencial explícita | passe `-SenhaRemota` via `Get-Credential`/secret manager; sem ela o script aborta |
| `-Ambiente` inválido | valor fora de `HML\|PROD` | use `-Ambiente HML` ou `-Ambiente PROD` |
| publish em PROD aborta antes do build | trava: sem `-Backup 1` ou sem digitar `PROD` | publique com `-Backup $true` e confirme digitando `PROD` |
| pre-flight aborta: `app_offline.htm` residual | deploy anterior interrompido | resolva manualmente no servidor antes de republicar |
| pre-flight aborta: `appsettings` ausente no servidor | configuração do ambiente não criada | crie o `appsettings` do ambiente no servidor antes do publish |
| pre-flight aborta: `runtime-config.json` ausente no servidor | primeiro deploy sem o arquivo no servidor | crie `assets\config\runtime-config.json` no servidor com a `apiBaseUrl` do ambiente antes do publish |
| script não conecta ao servidor | rede/firewall/VPN | valide rota até o servidor IIS do ambiente (`192.168.2.130` HML, `192.168.2.254` PROD; SQL Server é `192.168.2.154`) |
| autenticação no `c$` falha (fase 5) | credencial/permissão — duas causas distintas: sessão `net use` residual ou senha do usuário de IIS rotacionada | diagnóstico por mensagem em [`09-TROUBLESHOOTING.md`](../troubleshooting.md); obtenha credencial segura e revise a permissão; não registre a senha |
| backend fica offline | mirror falhou ou `app_offline.htm` permaneceu | conclua o mirror; remova o offline somente após validar artefatos |
| robocopy código 11 | DLL ainda bloqueada pelo worker | aguarde 1–2 min e repita; não remova o arquivo de offline antes do mirror |
| logout redireciona, mas retorna 401 | interceptor exclui `/auth/` do Bearer | limpar estado local não prova revogação; trate o token como ativo e solicite correção |
| Chat/Conhecimento 404 ou HTML da SPA | reverse proxy/ARR ausente ou regra não encaminha as novas rotas | confirme `/api/chat/*` e `/api/conhecimento/*` chegando ao backend `:1009`; não há fallback mock no contrato atual do Chat |
| Chat retorna 403 | operador sem função ativa ou função sem mapeamento de workspace habilitado | confirme operador/função e `tbchat_workspace_funcao`; o workspace não é fornecido pelo cliente |
| Chat retorna 502/504 | AnythingLLM ou Claude sem configuração/acesso, erro do provedor ou timeout | teste conectividade a partir do backend e configuração efetiva sem imprimir chaves; não espere resposta substituta mock |
| Conhecimento retorna 503/403 ou OAuth falha | configuração Gitea ausente, revisor não associado, credencial pessoal não conectada ou token de serviço ausente | confira configuração protegida, vínculo de função e estado de conexão; nunca copie tokens para logs |
| Sincronização pendente/erro | VM/serviço `jca-sync-trigger`/gatilho inacessível ou `KnowledgeSync` não configurado | verifique `jca-sync-trigger.service` (porta 8090) e logs na VM, endpoint `/api/conhecimento/sincronizacao` e firewall restrito à Central |
| frontend 404 em rota SPA | URL Rewrite/web.config | publique o pacote correto e valide o módulo IIS |
| backend 500 | ambiente, banco, configuração ou `web.config` | leia stdout log/event log, recicle e compare a configuração efetiva |
| **login em 500 depois do publish, com `CARGO_ID` já presente** | o modelo publicado também mapeia **`CARGO_Nivel`/`CARGO_FuncaoId`** e o banco ainda não tem essas colunas. O `preflight-schema.ps1` **hoje** verifica as duas, mas não cobria no incidente de árvore não commitada em **PROD, 2026-10-27** | faça backup (`dbCENTRAL_OPERACAO_pre_permissoes_20261027-233801.bak`) e aplique `Permissoes_Idempotente.sql` e depois `Permissoes_Matriz_Idempotente.sql`; confirme `COL_LENGTH('dbo.tbcargo','CARGO_Nivel')` e repita o login fictício (**401**). Diagnóstico em [`troubleshooting.md`](../troubleshooting.md) §20 |
| **login em 500 só para usuário válido** (fictício continua `401`), com `InvalidCastException` `Byte -> Int32` | tipo do CLR ≠ tipo da coluna: `tbcargo.CARGO_Nivel` é `tinyint` e `Cargo.Nivel` estava como `int`, então o `SqlClient.GetInt32` estoura ao materializar o `Cargo`. Nem `validate` nem o preflight checam tipo | mapeie `Cargo.Nivel` como `byte` (sem DDL: a coluna continua `tinyint`); confira `DATA_TYPE` em `INFORMATION_SCHEMA.COLUMNS`. Corrigido em `059183d`/`a7d3447`. Diagnóstico em [`troubleshooting.md`](../troubleshooting.md) §20 |
| smoke falha imediatamente | IIS ainda reciclando | aguarde 30–60 s e rode novamente |
| smoke passa, negócio falha | escopo insuficiente do script | faça login e QA manual |
| 500 só no módulo Comercial do Dashboard | rota duplicada entre `Controllers/` e `Controllers/Dashboard/` | confirme que cada prefixo de rota existe em um único controller; ver seção 12 de [`09-TROUBLESHOOTING.md`](../troubleshooting.md) |
| 500 "Invalid column name" no Administrativo ou Financeiro | coluna inexistente mapeada pelo espelho legado | valide `INFORMATION_SCHEMA.COLUMNS` de `tbcontrato`; note que `DIA_FATURAMENTO` é `smallint` e `QTDE_LICENCA` também |
| 500 "Sintaxe incorreta próxima a '$'" | `OPENJSON` gerado pelo EF em banco com `compatibility_level 100` | filtre em memória; o InMemory não reproduz o erro |
| tela do Kanban comercial sem o botão "Viu/Não viu" | `LeadContatoVisualizado` não aplicada no banco alvo | aplique `Migrations/Sql/LeadContatoVisualizado_Idempotente.sql` antes de publicar |
| 500 "Invalid column name 'LEAD_Arquivado'/'LDC_NomeContato'/'OPP_ProdutoId'" no Comercial | `ComercialFunilProdutos` não aplicada no banco alvo | aplique `Migrations/Sql/ComercialFunilProdutos_Idempotente.sql` antes de publicar; confira também `OBJECT_ID('tbproduto')` |
| 500 `Invalid object name 'tbclienteacesso'` / `Invalid column name 'AAC_*'` em `/api/v1/acessos` | `AcessosEmpresas` não aplicada no banco alvo | aplique `Migrations/Sql/AcessosEmpresas_Idempotente.sql`; sem ela as 32 rotas do módulo caem |
| 500 `Invalid object name 'tbgrupoacesso'` / grupo vazio / erro de FK em `CAO_GrupoId` | `AcessosGrupoClienteErp` não aplicada no banco alvo (a tabela deve não existir mais e a FK aponta para `tbcliente_grupo`) | aplique `Migrations/Sql/AcessosGrupoClienteErp_Idempotente.sql` imediatamente antes do publish; ver a seção da migration |
| erro ao criar o índice único de `CAO_Cnpj` (ou de qualquer coluna filtrada) | `CREATE INDEX` filtrado exige os `SET` de sessão do padrão | rode o script como está; no SSMS, habilite as opções em *Query Options*. ⚠️ Só cabe hoje em scripts legados: `IX_tbclienteacesso_CAO_Cnpj` não existe mais (drop pela `20261002161552`) |
| 500 `Invalid object name 'tbsetor'` / `'tbbloqueio'` ou `Invalid column name 'TRF_SetorId'` em `/implantacao/kanban/board` | `KanbanGeral` não aplicada no banco alvo | aplique `Migrations/Sql/KanbanGeral_Idempotente.sql` **antes** de publicar; sem ela caem as 13 rotas do Kanban Geral |
| 500 `Invalid object name 'tbfuncaoetapa'` ou `Invalid column name 'TRF_FuncaoId'` / `'PRJ_FuncaoId'` / `'CLK_FuncaoId'` em `/implantacao/kanban/board` | **Fase 0** da conversão Setor → Função não aplicada | rode `Migrations/Sql/SetorParaFuncao_Idempotente.sql` **antes** de publicar; ele é aditivo e roda com o código antigo no ar |
| quadro inteiro fora do ar depois de rodar script de consolidação | **Fase 3** aplicada com o código antigo ainda publicado | publique o código novo e rode a Fase 3; com o código novo no ar, a consolidação é o passo esperado — o inverso é que quebra |
| `THROW` 50001–50004 ao rodar a consolidação | a Fase 0 não foi aplicada | rode `Migrations/Sql/SetorParaFuncao_Idempotente.sql` primeiro; o script se protege de propósito |
| coluna "BLOQUEADO" de volta no quadro | coluna ainda ativa no banco alvo | o script desativa (`CLK_Ativa = 0`); reaplique o script e confira `CLK_Ativa` |
| `tbfuncao` sem `FUNCAO_ID = 4` ("Financeiro") | script de cadastro não rodado | rode `Migrations/Sql/FuncaoFinanceiro_Idempotente.sql`; é dado, não migration. A API do quadro depende de `tbfuncao` existir, mas a função 4 só é necessária se houver conteúdo financeiro |
| `403` ao mover card ou abrir bloqueio | card de outra função e operador fora de `GESTOR_MASTER`/gestão da função | regra de `FuncaoAuthorizationService`; conferir `FUNCAO_ID`/`CARGO_ID`, `tbgestor` e `TRF_FuncaoId`/`PRJ_FuncaoId` do card |
| backend não sobe ao publicar após ligar `CifradorAesGcm` | `Acessos:ChaveCifra` ausente, não-base64 ou ≠ 32 bytes | o construtor lança; reinstale a chave válida no servidor e recicle o pool |
| `eventosAuditoriaPerdidos` > 0 em `/acessos/estado-cifra` | fila de auditoria cheia ou processo encerrado com pendências | o contador zera ao reiniciar: anote antes, investigue o `LogError` e considere manter uma instância do backend |
| 500 `Invalid object name 'tbwhatsapp_atendimento'` (ou `tbwhatsapp_numero`, `tbwhatsapp_mensagem`, …) em `/api/v1/whatsapp/*` | `WhatsApp_Idempotente.sql` não aplicada no banco alvo | aplique `Migrations/Sql/WhatsApp_Idempotente.sql` **antes** de publicar (em **lotes**, sem o `GO` no mesmo comando — §5.1 do troubleshooting); sem as 7 tabelas, todas as rotas do módulo caem. **Já aplicada em `dbBUSINESS_HML` em 2026-10-21** |
| **todas as rotas do backend respondem `500` com corpo vazio** (inclusive `/naoexiste`, que deveria ser `404`) | schema errado do `<environmentVariables>` no `web.config` do backend (o bloco precisa ser **filho** de `<aspNetCore>`, com `<environmentVariable name= value=>`): o ANCM não sobe o processo | corrija o bloco, recicle (`app_offline.htm` ou recycle do app pool) e rode o `smoke.ps1` — o check **“Swagger/API”** é o detector confiável. Diagnóstico em [`09-TROUBLESHOOTING.md`](../troubleshooting.md) §13.1 |
| **módulo de WhatsApp cai depois do publish** (webhook em `503`, `502` no envio) | o bloco `<environmentVariables>` do `web.config` foi sobrescrito pelo publish (`/MIR` exclui só `appsettings*.json` e `app_offline.htm`) | **reaplicar** o bloco `WhatsApp__*` no `web.config` do servidor **depois** do publish e **recyclar**; o `smoke.ps1` **não** detecta isso (o módulo não é coberto pelo script) — a verificação é o `GET /api/v1/whatsapp/saude` |
| webhook do WhatsApp sempre em `503` | módulo desabilitado (`WhatsApp:Habilitado = false`) ou `WhatsApp:WebhookSecret` vazio no servidor | ligue o módulo por variável de ambiente (`WhatsApp__Habilitado=true`) e defina o segredo; o `503` é proposital (o gateway repete a entrega). Diagnóstico completo em [`09-TROUBLESHOOTING.md`](../troubleshooting.md) §19 |
| **erro 8152** ("dados seriam truncados") ao gravar o status do número, com a linha da fila **presa em "processando"** e o **telefone do número nunca gravado** | o **script de schema não foi reaplicado** depois do deploy: `tbwhatsapp_numero.WAT_Status` continua `varchar(20)` e o código grava o status canônico | **reaplicar** `Migrations/Sql/WhatsApp_Idempotente.sql` no banco do ambiente (ele tem o **bloco idempotente de ajuste de largura**, que alarga a coluna para `varchar(50)` sem recriar a tabela). Diagnóstico em [`09-TROUBLESHOOTING.md`](../troubleshooting.md) §19.10.1 |
| script mostra caminho do backend duplicado | o resumo do deploy antepõe `inetpub\wwwroot` a um destino que já o contém | use os caminhos reais desta runbook e valide o sistema de arquivos; é falha de apresentação, não do mirror |

Para locking local e remoto, consulte [`09-TROUBLESHOOTING.md`](../troubleshooting.md).

