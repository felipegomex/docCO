---
id: auditoria-seguranca
titulo: "Auditoria de segurança e prontidão para produção"
tipo: operacao
area: seguranca
sistema: central
tags: [auditoria, seguranca, riscos]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - scripts
testes:
  - docs/
tokens_estimados: 3347
ficha: docs/modulos/auth-rbac-sistema.md
---
> Ficha: [ficha](../modulos/auth-rbac-sistema.md).

# Auditoria de segurança e prontidão para produção

**Data:** 2026-10-20  
**Escopo:** análise estática do repositório Central de Operação. Não foram consultados o IIS, variáveis de ambiente, banco de produção ou resultados de testes de carga; build e testes automatizados não foram executados nesta auditoria.

## 1. Veredito Geral de Prontidão

**Nota: 3/10 — não considero o sistema pronto para produção sem corrigir os bloqueios abaixo.**

O repositório mostra medidas pontuais positivas — por exemplo, access token mantido em memória no frontend, refresh token em cookie `HttpOnly`, rotação de refresh token e tickets de uso único para operações sensíveis. Porém, há riscos relevantes de autenticação, exposição de credenciais e transporte inseguro.

**Contexto identificado no código:** sistema interno de conhecimento e operação; frontend Angular 18 e API ASP.NET Core 8, com EF Core e SQL Server. Há módulos de implantação, agenda, dashboard comercial/administrativo, exploração de banco, acessos de empresas e integração RAG/AnythingLLM. A escala esperada não foi informada. O sistema trata credenciais de acesso remoto e de sistemas de clientes, além de dados comerciais e operacionais.

## 2. Achados Críticos (Problemas graves que precisam ser corrigidos antes de qualquer deploy)

1. **P0 — Chave JWT conhecida pode ser aceita em produção.**  
   `backend/Central_BackEnd/appsettings.json` contém um valor placeholder para `Jwt:Key`. `JwtKeyProvider` (`Services/JwtKeyProvider.cs`) identifica esse valor, mas apenas registra um erro: a aplicação não interrompe a inicialização. Se a variável `JWT_KEY` não estiver configurada no servidor, um atacante que conheça o placeholder pode forjar tokens, inclusive com perfil de administrador.  
   **Impacto:** comprometimento total da autenticação e possível acesso administrativo.

2. **P0 — Senhas de operadores armazenadas em formato inadequado.**  
   `AppDbContext.cs` mapeia `tboperador.SENHA` como `varchar(10)`. `SegurancaHelper.SenhasIguais` aplica SHA-256 tanto à senha armazenada quanto à informada para comparar em tempo constante; isso não transforma a senha armazenada em hash.  
   **Impacto:** quem obtiver uma cópia do banco obtém as senhas diretamente; o limite de 10 caracteres restringe ainda mais a segurança. Senhas eventualmente reutilizadas em outros serviços também ficam expostas.

3. **P1 — Autorização insuficiente para revelar credenciais de empresas.**  
   `AcessosController` exige autenticação e prova recente da senha do próprio operador, mas seu próprio comentário declara que qualquer operador autenticado pode usar o módulo. Não identifiquei ali uma restrição de perfil para consultar ou revelar credenciais de RDP, VPN, AnyDesk e banco. A prova de senha reduz uso casual, mas não limita o conjunto de usuários autorizados.  
   **Impacto:** qualquer conta interna comprometida ou operador com acesso legítimo pode revelar credenciais de clientes, com potencial acesso a redes e sistemas externos.

4. **P1 — Cookie de refresh sem `Secure` e CORS com origens HTTP.**  
   Em `AuthController.cs`, as cookies de refresh são criadas com `Secure = false`. `appsettings.json` permite origens HTTP, inclusive endereços internos, e `Program.cs` habilita credenciais no CORS. Também não há `UseHttpsRedirection()` no pipeline da aplicação.  
   **Impacto:** quando essas origens e rotas HTTP estão acessíveis na implantação, as credenciais de sessão ficam sujeitas a interceptação ou abuso por origens permitidas comprometidas. `HttpOnly` protege contra leitura direta por JavaScript, mas não substitui transporte HTTPS.

5. **P1 — O chamador escolhe o workspace do RAG sem autorização por workspace.**  
   `RagProxyService.ChatAsync` usa o `WorkspaceId` recebido no corpo da requisição (`Models/RagChatRequest.cs`) e o encaminha ao AnythingLLM. O endpoint exige JWT, mas não valida se aquele operador pode acessar o workspace solicitado.  
   **Impacto:** se a credencial de serviço do AnythingLLM alcançar múltiplos workspaces, um usuário autenticado pode tentar consultar conteúdo de outras áreas.

6. **P1 — A configuração versionada não garante inicialização segura do módulo de acessos nem a migração dos dados legados.**  
   `Program.cs` registra `CifradorAesGcm`, mas o `appsettings.json` contém um placeholder inválido para `Acessos:ChaveCifra`. Sem a variável de ambiente esperada, a resolução do serviço falha quando o módulo é acessado, sem validação antecipada na inicialização. Além disso, `CifradorAesGcm.Decifrar` devolve como texto puro qualquer valor sem prefixo `v1:`. Assim, ativar AES não prova que as credenciais já existentes foram cifradas.  
   **Impacto:** falha em produção se a chave não estiver provisionada e, se o backfill não tiver sido confirmado, credenciais antigas permanecem legíveis apesar de a implementação ativa indicar AES.

## 3. Achados Moderados e de Baixo Risco

- **Token de acesso válido por quatro horas.** `Jwt:AccessTokenMinutes` está configurado como `240`. Desativar um operador ou remover seu perfil não invalida os tokens já emitidos; a API pode continuar aceitando as claims antigas até a expiração. (`AuthService.cs`, `Program.cs`.)

- **Bloqueio de conta pode ser usado para negação de serviço e crescer em memória.** `BruteForceGuard` bloqueia por nome de usuário após cinco falhas e mantém um dicionário local. Usuários legítimos podem ser bloqueados por tentativas deliberadas, e nomes aleatórios podem acumular entradas; a limpeza ocorre quando a mesma chave volta a ser consultada. (`Services/BruteForceGuard.cs`.)

- **Limitação de requisições não cobre todos os caminhos custosos.** O proxy RAG não tem política de rate limit aplicada, assim como a maioria dos endpoints de leitura do explorador de banco. As políticas configuradas são locais ao processo, não compartilhadas entre instâncias.

- **Explorador de banco expõe metadados sem autorização por função.** `DatabaseController` exige apenas autenticação. Há endpoints que retornam configuração do banco, estado da conexão, metadados e corpo de procedures/triggers. A resposta de status também pode incluir `ex.Message` (`TestarConexaoAsync`). A conexão do explorador não habilita criptografia por padrão (`DatabaseConnectionService.cs`).  
  **Impacto:** exposição desnecessária de arquitetura, lógica SQL e detalhes de infraestrutura para qualquer usuário autenticado; o impacto depende da política interna de acesso esperada.

- **Listagens sem paginação e consultas potencialmente pesadas.** `ComercialService.ListarLeadsAsync` materializa todos os leads, oportunidades e contatos antes de agregar os dados. Já `DatabaseMetadataService.ListarTabelasAsync` executa uma contagem de registros para cada tabela listada. Com crescimento de dados, essas operações podem elevar latência, alocação de memória e carga no SQL Server.

- **Consultas N+1 no detalhe de tarefas.** `TarefaService.ObterAsync` consulta individualmente cada chamado legado associado à tarefa. Tarefas com muitos chamados geram múltiplas idas ao banco.

- **Auditoria funcional pode aceitar identidade informada pelo cliente em alguns campos.** Em `TarefasController`, campos como `CriadorId` e `UsuarioAlteracao` são preenchidos com a identidade do token apenas quando o corpo da requisição os deixa vazios. A autorização usa o operador autenticado, mas os campos de autoria podem ficar sujeitos a falsificação.

- **Swagger está habilitado na configuração versionada.** `SwaggerEnabled` está `true` em `appsettings.json`; `Program.cs` publica a documentação quando essa configuração está ativa. É necessário confirmar que a implantação de produção sobrescreve o valor.

- **Documentação de criptografia diverge do registro atual de DI.** `docs/decisoes/002-criptografia-desligada.md` descreve `CifradorEmBranco` como implementação ativa, enquanto `Program.cs` registra `CifradorAesGcm`. Essa divergência prejudica a operação e a confirmação de que o backfill foi realizado.

## 4. Recomendações e Plano de Remediação Priorizado

| Problema | Impacto | Passo exato para corrigir |
|---|---|---|
| Chave JWT placeholder aceita | **P0 —** emissão de tokens forjados e bypass de autenticação | Exigir `JWT_KEY` forte no ambiente do servidor, rejeitar placeholders e falhar a inicialização se a chave estiver ausente ou fraca. Rotacionar a chave se o placeholder já tiver sido usado em uma implantação acessível. |
| Senhas de operadores em texto recuperável e limitadas a 10 caracteres | **P0 —** comprometimento de contas após vazamento do banco | Criar migração para armazenamento com hash de senha salgado e resistente a força bruta, usando `PasswordHasher` do ASP.NET Identity ou Argon2id/PBKDF2 com parâmetros atuais. Planejar reset ou migração controlada das credenciais existentes; remover o limite de 10 caracteres. |
| Acesso amplo às credenciais de clientes | **P1 —** exposição de acessos de terceiros | Definir perfis ou políticas por função e escopo de cliente para visualizar, editar e revelar senhas. Validar a autorização no backend em cada endpoint e auditar operador, cliente, finalidade e resultado sem registrar o segredo. |
| Cookies sem `Secure` e origens HTTP | **P1 —** exposição ou abuso de sessão em redes inseguras | Em produção, definir `Secure = true` para cookies, restringir CORS a origens HTTPS exatas e habilitar redirecionamento HTTPS no pipeline ou garantir essa política no IIS com teste de ponta a ponta. |
| RAG aceita workspace enviado pelo usuário | **P1 —** leitura de conteúdo fora do escopo autorizado | No backend, mapear operador/perfil para uma lista permitida de workspaces e rejeitar qualquer `WorkspaceId` não autorizado. Aplicar limites de tamanho e rate limit por usuário ao endpoint de chat. |
| Estado global de AES não comprova backfill | **P1 —** credenciais antigas podem continuar em texto puro | Consultar/inventariar todas as linhas legadas, executar e validar o backfill de modo idempotente, testar decifragem e falhar se houver registros sem prefixo válido. Fazer o endpoint de estado indicar a situação real dos dados, não apenas a classe registrada. |
| Tokens longos e sem revogação imediata | **P2 —** privilégios antigos persistem após alteração de conta | Reduzir a duração do access token e adicionar mecanismo de revogação/versionamento de sessão ou validação de estado do operador para operações sensíveis. |
| Rate limit e bloqueio em memória | **P2 —** bloqueio indevido, abuso distribuído e pressão de memória | Limitar e expurgar entradas, combinar quotas por IP e conta, evitar bloqueio permanente de alvo por falhas de terceiros e usar armazenamento distribuído quando houver múltiplas instâncias. |
| Endpoints do explorador de banco sem escopo e consultas sem limites | **P2 —** exposição de metadados e carga no banco | Restringir o controller a um perfil operacional específico, aplicar rate limits às leituras, limitar tamanho de resultados e configurar criptografia com validação de certificado na conexão. |
| Listagens completas e consultas N+1 | **P2 —** degradação com crescimento de dados | Adicionar paginação e projeções para leads; fazer agregações no SQL; substituir consultas individuais de chamados por consulta em lote; revisar planos e índices das consultas de contagem do explorador. |
| Autoria fornecida pelo cliente e documentação divergente | **P2 —** trilha de auditoria imprecisa e operação inconsistente | Sobrescrever sempre os campos de autoria com a identidade do JWT e atualizar os documentos para refletir o registro e a migração de cifra efetivamente vigentes. |

## 5. Sugetiões de melhoria arquitetural a longo prazo

- **Centralizar identidade e autorização:** considerar OIDC/SSO corporativo com MFA e políticas explícitas baseadas em perfil, função e recurso. Evitar depender de roles embutidas em tokens de longa duração para decisões que precisam ser revogadas rapidamente.
- **Adotar gestão dedicada de segredos:** usar cofre de segredos/KMS para chaves JWT, cifra e integrações; planejar rotação de chaves e versionamento de envelopes. Para AES-GCM, avaliar AAD para vincular ciphertext ao registro e ao tipo de credencial.
- **Isolar o explorador de banco:** usar credencial estritamente somente leitura, limitar acesso de rede, restringir quais schemas podem ser inspecionados e separar esse acesso das credenciais usadas pela aplicação.
- **Tornar o RAG uma fronteira de autorização própria:** resolver workspace no servidor, aplicar quotas e limites de contexto, e definir regras de tratamento para dados sensíveis antes de enviar conteúdo a um provedor externo.
- **Fortalecer observabilidade e resiliência:** substituir escrita direta em console por logging estruturado; adicionar métricas de latência, falhas, quotas e consultas lentas, sem registrar senhas, tokens ou conteúdo sensível.
- **Instituir verificação contínua:** complementar os testes unitários existentes com testes de integração para autorização, revogação, rotação de refresh token e isolamento entre workspaces; incluir análise de dependências, SAST e testes de carga antes de releases relevantes.
