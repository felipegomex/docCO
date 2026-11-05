---
id: regras-negocio.22
titulo: "Dados sensíveis"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2065
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Dados sensíveis

**Situação: parcial; há controles, mas a superfície de dados é ampla.**

- **Acessos Empresas é um módulo CRUD no `AppDbContext`** (tabelas `tbclienteacesso` e as 5 de acesso; os **grupos** vêm da legada `tbcliente_grupo` e as **empresas** da legada `tbcliente`); **não** há mais integração com Google Sheets, nem serviço, seção de configuração ou projeção em memória dessa origem. A antiga `tbgrupoacesso` foi dropada.
- `GET /api/v1/acessos/clientes` devolve paginação com flags e booleanos, **sem credencial**. O detalhe (`GET /clientes/{id}`) traz as 5 listas de acesso com `temSenha: boolean` — nunca o valor.
- A **única** forma de a senha sair do servidor são `GET /rdp/{id}/senha` e `GET /vpn/{id}/senha`. Cada chamada enfileira `REVEAL_SENHA` **antes** de devolver, mesmo quando o acesso não tem senha — o fato de alguém ter tentado ver fica registrado. A senha em si **nunca** é gravada na auditoria; o log carrega tipo, id do acesso, id do cliente, usuário, IP e navegador.
- **A criptografia está ativa no DI:** `CifradorAesGcm` grava as senhas novas com AES-256-GCM. `CifradorEmBranco` permanece apenas como fallback de teste e não está registrado. `Decifrar` aceita linhas antigas sem o prefixo `v1:`, então a cobertura do backfill precisa ser contada — `GET /api/v1/acessos/estado-cifra` (`implementacao`, `ehCifradoDeVerdade`, `eventosAuditoriaPerdidos`) mostra a implementação, não a cobertura.
- Mitigações já em vigor enquanto a cifra não é liberada: coluna `*_SenhaCifrada nvarchar(400)` **já nasce dimensionada** para o envelope AES-256-GCM (por isso ligar a cripto depois **não** exige `ALTER TABLE`), abstração `ICifrador` no DI, aviso no log, auditoria completa e aviso na interface. Procedimento de liberação em 4 passos — gerar a chave, gravá-la no servidor, trocar o registro em `Program.cs` e **fazer o backfill** — em [`04-ESTRUTURA-DADOS.md`](../../arquitetura/banco-dados.md).
- ⚠️ **Criptografia ligada sem backfill = aparência de segurança sem segurança.** Sem o backfill, as linhas antigas seguem em texto puro e `Decifrar` cai no caminho de compatibilidade (valor sem prefixo `v1:`) e devolve o texto puro **silenciosamente** — a tela para de avisar e `ehCifradoDeVerdade` vira `true`, mascarando metade do histórico em claro. O backfill não é opcional nem adiável.
- `CifradorAesGcm` recusa **marcadores de placeholder** (`TROQUE_POR_UMA_CHAVE`, `DEV_CHAVE_NAO_USAR`, `CHANGEME`, `CHANGE_ME`) antes de tentar decodificar base64, com mensagem que aponta `GerarChaveBase64()`. Sem isso o placeholder versionado — que contém `_` e portanto nem é base64 — produziria um erro enganoso de "base64 inválido". `appsettings.Development.json` é **ignorado pelo git** (a chave de dev nunca foi versionada); `appsettings.json` **é** versionado e mantém o placeholder de propósito.
- ⚠️ **`tboperador.SENHA` é `varchar(10)` em texto claro e a cifra nova NÃO cobre isso.** O comparador (`SegurancaHelper.SenhasIguais`) hasheia **os dois lados**, então o valor gravado equivale à senha: ler a linha da tabela já dá capacidade de login, e a coluna trunca em 10 caracteres. É um trabalho separado, ainda não agendado no código.
- Falha de auditoria é registrada no log (`Error`), mas não impede a resposta nem desfaz a operação de negócio.
- O Database Explorer usa uma conexão SQL separada, configurada por variáveis de ambiente ou seção `DatabaseExplorer`. O DTO de configuração mascara a senha; os endpoints exigem a policy `perm:aba.ferramentas.banco-dados:Ler`; endpoints de procedimentos e gatilhos podem devolver o corpo completo a quem tem a policy.
- **Chat:** página `/chat` (`ChatIframeComponent`) — iframe para o AnythingLLM autenticado por `GET /api/chat/sso`; sem API REST de agentes na Central (contrato removido no `cb36ad3`). O conteúdo da conversa é processado pelo AnythingLLM e pelos provedores configurados nele; a Central não grava pergunta/resposta. Segredos do AnythingLLM, Claude e Gitea ficam na configuração do servidor/VM.
- **Conhecimento:** acesso de revisão é associado por administrador a operador/função; usuários não autorizados são barrados também no backend, independentemente do guard Angular. A conta Gitea é pessoal por revisor e seus tokens são cifrados em `tbconhecimento_credencial_gitea`. O token de serviço prepara branch/alterações/PR e o revisor aprova/mescla com OAuth pessoal, tanto na aprovação quanto na rejeição. Rejeitar exige motivo e mantém o documento no mesmo caminho com `status: rejeitado`; pré-requisito operacional: `_config/status-documento.yml` no repositório Gitea deve aceitar `rejeitado`. Um `409` de SHA contém SHA e conteúdo atuais, para comparar ou recarregar pela UI. `tbconhecimento_auditoria` registra ações e metadados do documento, incluindo motivo e IP.
- **Sincronização Conhecimento:** admin pode solicitar execução em `POST /api/conhecimento/sincronizar` e consultar estado em `GET /api/conhecimento/sincronizacao`; o serviço da VM atende `POST /api/sync` e `GET /api/status`. `01-empresa/comercial/` vai somente ao workspace comercial. A primeira sincronização tenta criar workspaces ausentes se a API AnythingLLM suportar, mas não cria vínculos de revisores; a migration/SQL semeia `tbchat_workspace_funcao` (funções 1, 2, 3 e 5), mas não revisores.
- **Atendimento WhatsApp:** a chave da API do gateway (`WhatsApp:ApiKey`) e o segredo do webhook (`WhatsApp:WebhookSecret`) ficam **só no servidor** (variáveis `WhatsApp__ApiKey` / `WhatsApp__WebhookSecret`); o `appsettings.json` versionado guarda só o esqueleto (`"ApiKey": ""`, `"WebhookSecret": ""`) e o módulo nasce **desabilitado**. O webhook é autenticado por HMAC-SHA256 do corpo cru, com comparação em tempo constante. A **conversa** é dado sensível: fica em `tbwhatsapp_mensagem` (inclusive o texto) e o histórico append-only em `tbwhatsapp_evento`; o **conteúdo do QR Code** nunca vai para log nem para a documentação. A porta do gateway (`2785`) não é exposta publicamente: é liberada **apenas** para os dois servidores da Central, e a `1009` de cada servidor da Central é liberada **apenas** para a VM do gateway (`192.168.2.253`).
- Segredos de AnythingLLM, Claude, OAuth Gitea, token de serviço Gitea e gatilho de sincronização pertencem à configuração protegida do servidor/VM; não devem estar no repositório. Ver [`troubleshooting.md`](../../operacao/troubleshooting.md) e [`deploy.md`](../../operacao/deploy.md) para limitações operacionais.
- Há valores de desenvolvimento preenchidos em [appsettings.Development.json](../../../backend/Central_BackEnd/appsettings.Development.json), incluindo chaves de configuração sensíveis. Este documento não os reproduz; mova-os para ambiente/user-secrets e rotacione qualquer valor real usado.

