# AGENTS.md — Central de Operação

## Idioma
- Responder sempre em **português (PT-BR)**, salvo pedido explícito do usuário por outro idioma.

## Prioridades
1. Concluir a tarefa solicitada.
2. Preservar a integridade do código.
3. **Atualizar a documentação automaticamente** (§ Documentação).
4. Manter o contexto leve (§ Protocolo e orçamento).

## Invariantes
- `docs/` é a fonte de verdade da Central. Gitea, AnythingLLM e RAG são outro corpus e não entram no trabalho de documentação da Central.
- Monorepo: `frontend/` (Angular 18), `backend/` (ASP.NET Core 8), `docs/` e `scripts/`. Branch de desenvolvimento: `developer`.
- Não alterar schema, migration ou banco sem pedido explícito. Não commitar nem fazer deploy sem pedido explícito.
- Segurança: consultas EF Core parametrizadas, sem SQL interpolado; mensagens genéricas ao cliente e detalhe só no log; nunca expor token, cookie, chave ou senha.
- Em divergência entre documento e código, **pare e pergunte**; não invente comportamento.

## Protocolo de contexto (obrigatório)
1. Classifique a intenção: bug, feature, refactor, docs ou operação.
2. Leia [`docs/MAP.md`](docs/MAP.md).
3. Leia **uma** ficha de módulo em `docs/modulos/`.
4. Abra somente os arquivos/símbolos da seção **“Onde alterar”** da ficha.
5. Leia os testes citados pela ficha.
6. Se documento e código divergirem, pare e pergunte.
7. Responda com arquivo/símbolo, mudança, teste, risco e documento a atualizar.

## Orçamento de contexto
- Por tarefa: no máximo **2 fichas**, **4 trechos de código** e **12 tool calls**.
- Se precisar exceder, peça confirmação antes de continuar.
- Não abra documento profundo inteiro: use a seção citada pela ficha.
- Nunca abra stub (`status: dividido`); use o `INDEX.md` da subpasta.

## Contrato de saída
| Arquivo | Símbolo | Mudança | Teste |
|---|---|---|---|
| caminho do arquivo | função/classe/rota | o que muda | comando ou teste que cobre |

Inclua, após a tabela, **risco** e **documento a atualizar**.

## Quando parar e perguntar
- Arquivo, símbolo ou ficha inexistente.
- Divergência entre documentação e código.
- Requisito ambíguo ou conflito com política/invariante.
- Necessidade de segredo, schema, migration ou deploy não pedido.
- Orçamento de contexto estourado.

## Delegação
- Após alteração de código: delegue ao `docs-writer` a atualização de fichas/seções/índices.
- Conteúdo estático `*.data.ts`: delegue ao `content-editor`.
- O `docs-writer` edita seções, fichas e documentos canônicos; **nunca** edita stub (`status: dividido`).
- Revisão de segurança: delegue ao `security-auditor`.

## Pipeline Dev/Deploy
1. **Subir interno** para testar no navegador (dev, sem build).
2. **Validar** com `powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1` (segredos + build + testes). Se vermelho, **pare**.
3. `git add/commit/push` na `developer`.
4. Aplicar o SQL idempotente do commit no ambiente, quando houver.
5. **Deploy limpo**: empacotar, preflight de schema bloqueante, publicar no IIS e rodar smoke.

| Etapa | Comando |
|---|---|
| Dev local | skill `subir interno` |
| Gate | `scripts\validate.ps1` (skill `validar`) |
| Empacotar | `deploy.ps1 -BuildFrontend 1 -BuildBackend 1 -Publicar 0` |
| Publicar | `deploy.ps1 -BuildFrontend 0 -BuildBackend 0 -Publicar 1 -Backup 1` |
| Smoke | `scripts\smoke.ps1` |

- `deploy.ps1` chama `scripts/preflight-schema.ps1` e aborta se o schema do commit não estiver no banco.
- O smoke exige **401** no login fictício; **500 ali é schema/código quebrado**.
- Banco local: `appsettings.Development.json` → `Database:UseSqlServer` (`true` = homolog; `false` = InMemory).

## Segredos — nunca versionar
- Nenhum arquivo commitado pode conter senha, API key, token ou connection string com credencial.
- Use `user-secrets` ou variável de ambiente; no arquivo versionado, deixe placeholder vazio.
- Script de infraestrutura recebe credencial por **parâmetro obrigatório**, nunca constante.
- Antes de commitar, rode `scripts\validate.ps1`: o passo de segredos varre o índice do Git e é **bloqueante**.
- Se o gate acusar, mova o segredo para user-secrets/variável; não troque por outro segredo hardcoded.

## MCPs
- **Ferramentas nativas primeiro** (`read`, `grep`, `glob`, `git` local); MCP é complemento, não atalho.
- No máximo **3 tentativas** por ferramenta; se falhar **duas vezes**, pare e relate o que ficou pendente.
- Descrição de cada servidor em `.opencode/mcps/<nome>.md`; segredos em `.opencode/secrets/` (gitignored), nunca no `opencode.json`.
- Para ler um arquivo, buscar texto ou ver o histórico local, use a ferramenta nativa — não o MCP.

| MCP | Usar quando | Não usar quando | Cuidado |
|---|---|---|---|
| `sql-server` | schema, colunas, índices, dados de homologação | escrever/DDL ou consulta ampla sem filtro | read-only, limite de linhas, timeout |
| `github` | histórico, PR, commit, issue | código local | não substitui o `git` local |
| `context7` | documentação de biblioteca externa | documentação do projeto | pode custar tokens |
| `filesystem` | árvore/listagem ampla | ler arquivo único ou buscar texto | prefira `read`/`grep`/`glob` |

## Documentação (automática)
- Roteador: [`docs/MAP.md`](docs/MAP.md), gerado por `scripts/gen-docs-map.ps1`; não editar à mão.
- Fichas: `docs/modulos/*.md` (alvo ≤ 2 mil tokens). Elas **apontam** para o documento profundo; não duplicam conteúdo.
- Documentos profundos: `docs/arquitetura/`, `docs/referencia/` e `docs/operacao/`, divididos em seções com frontmatter. Abra a seção, não o documento inteiro.
- Ao concluir alteração relevante, delegue ao `docs-writer` a revisão dos documentos temáticos e dos READMEs afetados. Não pergunte se deve atualizar: faça.
- Novo componente/service/router, controller/endpoint/model/migration ou mudança de pipeline/deploy: siga o MAP e a ficha; o `docs-writer` atualiza o índice profundo, os cenários de QA e os READMEs afetados.
- Valide com `powershell -ExecutionPolicy Bypass -File .\scripts\check-docs.ps1 -ExigirFrontmatterSecoes`.
- Não duplique caminhos de módulo neste arquivo: consulte o MAP e a ficha.

## Regras de comunidade
- Não pergunte o que já foi definido; verifique e siga.
- Antes de criar branch, confirme se já existe no GitHub.
- Antes de taggear, confirme que a versão está mergeada em `master`.
- Um assunto, um arquivo; atualizar não é engordar. Substitua trechos obsoletos e preserve índices.
- Links apontam para **arquivos**, nunca para âncoras profundas.

## Convenção de commits
- `feat:` nova funcionalidade
- `fix:` correção de bug
- `docs:` apenas documentação
- `chore:` manutenção (deps, config)
- `refactor:` refatoração sem mudança de comportamento

## Observação
- Configuração/instruções (AGENTS.md, agentes, skills) valem em novas sessões; após modificá-las, informe o usuário para reiniciar o opencode.
