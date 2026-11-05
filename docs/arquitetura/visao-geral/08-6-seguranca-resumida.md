---
id: visao-geral.08
titulo: "6. Segurança resumida"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 959
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

## 6. Segurança resumida

- Access token JWT fica somente na memória do frontend.
- Refresh token fica no cookie HttpOnly `cc_refresh`; o JSON da API não entrega seu valor.
- Cada refresh rotaciona o token: o anterior é revogado e recebe um substituto.
- Refresh tokens expiram em quatro horas; cada uso válido cria uma nova janela de quatro horas, produzindo renovação deslizante durante a atividade.
- Login possui limitação de taxa e bloqueio local de tentativas; validação de dados sensíveis e leitura também possuem limites.
- CORS usa lista explícita de origens e `AllowCredentials`.
- Detalhes de Acessos exigem revalidação — a prova de senha é um ticket de acesso de 2 minutos, escopado à empresa, de uso único em cada escrita ou revelação — e geram auditoria. A senha só sai do servidor por dois endpoints de revelação (RDP e VPN), e cada chamada deixa registro `REVEAL_SENHA` com usuário, IP e navegador.
- **RBAC:** a permissão efetiva vem de `tbperm_cargo` (cargo) e `tbperm_operador` (exceção individual; a **negativa vence**), com escopo `Proprio`/`Funcao`/`Tudo`; `GESTOR_MASTER` tem bypass. As rotas protegidas usam policies `perm:<recurso>:<acao>`, e o **403** é auditado em `tbauditoriaimplantacao` (`ACESSO_NEGADO`). `tboperador.SENHA` continua em `varchar(10)` e a gestão de operadores **não exclui** — inativa (`SE_ATIVO = 'N'`).
- A auditoria do Chat guarda operador/workspace/resultado/data, sem pergunta, resposta ou fontes; pergunta e conteúdo recuperado são processados por AnythingLLM e Claude. Considere a transferência a provedores externos ao classificar dados enviados.
- A criptografia das credenciais de Acessos está ativa no DI (`CifradorAesGcm`, AES-256-GCM): escritas novas são cifradas. `CifradorEmBranco` permanece apenas como fallback de teste e não está registrado. `Decifrar` mantém compatibilidade com linhas antigas sem o prefixo `v1:`, então a cobertura do backfill precisa ser conferida contando essas linhas — `GET /api/v1/acessos/estado-cifra` mostra a implementação, não a cobertura.
- O Database Explorer não oferece execução de SQL arbitrário pelos endpoints atuais; trabalha com metadados, busca, relações, comparação de schemas e de corpos de procedures, construção de consultas e geração/validação estática de scripts de correção (somente texto, nunca executado).
- **Atendimento WhatsApp:** a chave da API do gateway e o segredo do webhook ficam **somente no servidor** (variáveis de ambiente `WhatsApp__ApiKey` / `WhatsApp__WebhookSecret`); o `appsettings.json` versionado guarda só o esqueleto. O webhook é autenticado por **HMAC-SHA256 do corpo cru**, com comparação em tempo constante, e a entrega é deduplicada por `X-OpenWA-Idempotency-Key`. O **QR Code** de conexão não vai para log nem para a documentação. A porta do gateway não é exposta publicamente: ela é liberada **apenas** para os dois servidores da Central, e a porta `1009` de cada servidor da Central é liberada **apenas** para a VM do gateway.
- Há limites relevantes: o cookie refresh usa `Secure=false`, os defaults operacionais de URL são HTTP e devem ser confirmados no ambiente, a comparação de senha usa SHA-256 sem algoritmo adaptativo com salt e `tboperador.SENHA` permanece `varchar(10)` guardando material equivalente à senha em claro — o comparador hasheia os dois lados, então ler a linha da tabela já permite autenticar — e o campo está **fora** do alcance da nova cifra.

A descrição completa dos controles, exceções e limites está em [`02-ARQUITETURA.md`](../componentes/INDEX.md).

