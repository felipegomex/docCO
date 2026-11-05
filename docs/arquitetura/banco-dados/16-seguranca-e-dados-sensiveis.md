---
id: sec.16-seguranca-e-dados-sensiveis
titulo: "Segurança e dados sensíveis"
tipo: referencia
area: dados
sistema: central
tags: [schema, banco, tabelas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 918
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/arquitetura/banco-dados.md](../banco-dados.md).

# Segurança e dados sensíveis

| Dado | Onde aparece | Tratamento confirmado / lacuna |
|---|---|---|
| Senha do operador | `tboperador.SENHA` (`varchar(10)`) e comparador de autenticação | ⚠️ **Material equivalente à senha em claro** — o comparador hasheia os dois lados, então o valor gravado autentica e ler a linha da tabela equivale a comprometimento. Sem hash adaptativo com sal e **fora** do alcance da nova cifra do módulo Acessos. |
| Senha de acesso de empresa | `RDP_/VPN_/ADK_/BCO_SenhaCifrada` | **Cifrada com AES-256-GCM** (`CifradorAesGcm` no DI). Linhas antigas sem o prefixo `v1:` continuam legíveis por compatibilidade até o backfill; a cobertura precisa ser contada. Só sai do servidor por `GET /rdp/{id}/senha` e `GET /vpn/{id}/senha`. |
| Refresh token | `tbrefreshtoken.TokenHash` e cookie `cc_refresh` | Banco guarda hash; cookie é HttpOnly, mas está com `Secure=false`. |
| IP/user-agent | `tbauditoriaacesso` e `tbauditoriaimplantacao` | Registrados para rastreabilidade; não substitui controle de acesso. |
| JSON antes/depois | `tbauditoriaimplantacao` | Serializado sem mascaramento explícito; pode carregar conteúdo operacional. **No módulo Acessos o payload é montado com campos escolhidos** (nunca inclui `SenhaCifrada`), e a troca de senha é registrada como observação "senha trocada — valor não registrado". |
| Senha do Explorer | `DatabaseExplorer:*` / `DB_EXPLORER_*` | Configuração mascarada no DTO; valores não são documentados aqui. |
| Definições de procedimentos/gatilhos | DTOs do Database Explorer | Podem ser completas e são acessíveis a qualquer usuário autenticado. |
| Segredo AnythingLLM | `AnythingLLM:ApiKey` | Consumido pelo backend para consulta dos workspaces; manter fora do repositório. |
| Segredo Claude | `Claude:ApiKey` (e `Claude:Model`, obrigatório) | Consumido pelo backend para gerar a resposta a partir das fontes; manter fora do repositório. |
| OAuth Gitea e token de serviço | `Gitea:OAuth:ClientSecret`, `Gitea:ServiceToken` | Segredos do servidor; credenciais OAuth pessoais dos revisores são cifradas em `tbconhecimento_credencial_gitea`. |
| Gatilho de sincronização | `KnowledgeSync:Secret` | Chave compartilhada entre Central e VM, configurada fora do repositório; nunca registrar seu valor. |
| Perguntas e respostas do Chat | AnythingLLM e Claude por HTTP; `tbchat_auditoria` | A pergunta é enviada aos provedores; a auditoria local registra operador, workspace, resultado e data/hora, sem persistir pergunta, resposta ou fontes. |
| **Chave da API e segredo do webhook do WhatsApp** | `WhatsApp:ApiKey` e `WhatsApp:WebhookSecret` (variáveis `WhatsApp__ApiKey` / `WhatsApp__WebhookSecret` no servidor) | **Nunca versionados**: o `appsettings.json` do repositório guarda só o esqueleto (`"ApiKey": ""`, `"WebhookSecret": ""`). O segredo do webhook é o **mesmo** valor registrado no gateway OpenWA. O **conteúdo do QR Code** nunca vai para log nem para este documento. |

O arquivo [appsettings.Development.json](../../../backend/Central_BackEnd/appsettings.Development.json) contém valores preenchidos de desenvolvimento, inclusive chaves e dados sensíveis. Nenhum valor é reproduzido aqui; antes de compartilhar o repositório, mover esses valores para ambiente/user-secrets e rotacionar os que forem reais.

