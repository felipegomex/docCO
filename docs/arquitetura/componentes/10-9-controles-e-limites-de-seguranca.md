---
id: componentes.10
titulo: "9. Controles e limites de segurança"
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
tokens_estimados: 1241
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

## 9. Controles e limites de segurança

### Controles presentes

- Guards Angular protegem o shell principal e a área administrativa; `[Authorize]` no backend é a fronteira de segurança.
- JWT valida assinatura, issuer, audience, algoritmo e expiração.
- O refresh token é rotativo, revogável e armazenado como hash.
- A policy de login limita requisições por IP: cinco por minuto fora de Development e cinquenta em Development.
- A policy de validação limita por usuário autenticado ou IP: cinquenta por minuto fora de Development e cem em Development.
- A policy de leitura limita a cem por minuto por usuário autenticado ou IP.
- Chat e Conhecimento têm policies próprias (12/60 requisições por minuto fora de Development e 30/120 em Development); como `UseRateLimiter()` antecede `UseAuthentication()`, a chave observada pode ser o IP.
- `BruteForceGuard` bloqueia a chave por quinze minutos após cinco falhas dentro de uma janela de cinco minutos.
- Acessos exige revalidação, limita a prova de senha a 2 minutos (uso único em `EDITAR`, escopada por empresa) e audita a visualização.
- Comparação de schemas tem limites de upload de 5 MB e 20 MB.
- Respostas de produção usam handler global e mensagens genéricas para erros não tratados.

### Limites que não devem ser ocultados

- `cc_refresh` está com `Secure=false`; qualquer tráfego HTTP pode expor a sessão à interceptação de rede.
- A chave JWT tem fonte única desde `57a7e4f`: `JwtKeyProvider` (singleton) resolve `JWT_KEY ?? Jwt:Key` uma vez e a mesma chave serve à emissão e à validação. O risco restante é operacional — `JWT_KEY` no ambiente do *application pool* é apagada a cada recycle do IIS, derrubando as sessões; use o ambiente da máquina.
- O comparador de senhas (`SegurancaHelper.SenhasIguais`) usa SHA-256 direto, sem salt e sem algoritmo adaptativo como Argon2, bcrypt ou PBKDF2 — e hasheia **os dois lados**: como `tboperador.SENHA` (`varchar(10)`) guarda a senha em claro, o valor guardado equivale à senha, e ler a linha da tabela já dá capacidade de login. A coluna trunca em 10 caracteres, então senha mais longa é cortada na gravação.
- O bloqueio de tentativas e o ticket de acesso são estado em memória do processo; não são armazenamento distribuído.
- Todos os endpoints do Database Explorer exigem autenticação, mas não exigem role `Administrador`; respostas de procedures e triggers podem incluir o corpo completo do objeto.
- A auditoria de Acessos registra falhas no log, mas a falha de auditoria não impede a resposta ao operador. Ela é **assíncrona por fila**, então um evento pode se perder se o processo encerrar com a fila cheia; o contador aparece em `GET /api/v1/acessos/estado-cifra`.
- O módulo Acessos Empresas tem `[Authorize]` **sem policy por papel**: qualquer operador autenticado cria, altera e inativa unidades e credenciais (grupos são somente leitura, vindos de `tbcliente_grupo`). A mitigação é a trilha de auditoria, não o controle de acesso. O RBAC já tem policies (`perm:<recurso>:<acao>`) e auditoria de 403, mas **ainda não foi aplicado ao módulo Acessos**.
- A gestão de operadores do RBAC **não exclui**: a saída é inativar (`SE_ATIVO = 'N'`). A senha continua limitada a 10 caracteres e é validada apenas por tamanho; o e-mail é validado por presença/posição de `@` e unicidade sem caixa, não por formato completo.
- As credenciais novas de Acessos Empresas são gravadas com `CifradorAesGcm`; linhas antigas sem o prefixo `v1:` continuam legíveis por compatibilidade e podem permanecer em texto puro até o backfill. O estado exibido em `GET /acessos/estado-cifra` informa a implementação registrada, não a cobertura do backfill.
- A nova cifra do módulo Acessos **não** cobre `tboperador.SENHA`.
- O guard estrito de `/admin` é do frontend; os controllers de projetos e tarefas têm `[Authorize]` sem role `Administrador`, enquanto escritas de tipos/colunas e o dashboard administrativo têm a restrição de role.
- O Chat envia pergunta e fontes a provedores externos (AnythingLLM e Claude); a auditoria local não inclui pergunta/resposta, mas isso não elimina o processamento externo do conteúdo.
- `AllowedHosts` está configurado como `*` nos arquivos versionados.
- `SwaggerEnabled` está `true` nos arquivos versionados; a configuração efetiva do servidor pode ser diferente porque os `appsettings*.json` são preservados no deploy.
- O `web.config` rastreado não define proxy de API; o encaminhamento relativo de Chat/Conhecimento depende de configuração IIS não visível no repositório.

A avaliação de risco deve comparar esses limites com a política interna antes de expor o sistema a redes diferentes da rede operacional.

