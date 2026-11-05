---
id: sec.09-4-database
titulo: "4. Database"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 3306
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 4. Database

### DB-01 — Abrir o Database Explorer e a Visão Geral

- **Objetivo:** Verificar o estado da conexão e entrar nas áreas de metadados do SQL Server.
- **Pré-condições:** operador autenticado; conexão configurada no ambiente.
- **Passos:** 1. Abrir `/database` ou `/database/visao-geral`. 2. Ler o banner de conexão e os contadores. 3. Usar a busca global do shell. 4. Abrir uma tabela, procedure, relationship ou diferença.
- **Resultado esperado:** o shell redireciona para Visão Geral, mostra conectado/desconectado e as seis abas ativas: Visão Geral, Explorador, Relacionamentos, Consultas, Diferenças e Configuração. A busca global consulta `GET /api/v1/database/search`; a Visão Geral mostra nove cards de metadados e o botão de testar conexão.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a disponibilidade dos metadados e o teste de conexão dependem do SQL Server configurado no ambiente.
- **Rotas/endpoints/tabelas:** `/database`, `/database/visao-geral`; `GET /api/v1/database/status`, `GET /api/v1/database/info`, `GET /api/v1/database/search`, todos `[Authorize]`; leitura de `sys.*` e demais metadados do SQL Server, sem escrita de negócio.
- **Status/automação:** **cenário documentado**; não há teste automatizado do shell ou da Visão Geral.

### DB-02 — Explorar tabelas, procedures e triggers

- **Objetivo:** Localizar objetos do banco e consultar sua estrutura ou definição.
- **Pré-condições:** operador autenticado; API de metadados disponível.
- **Passos:** 1. Abrir `/database/explorador`. 2. Pesquisar/expandir Tabelas, Procedures e Triggers. 3. Selecionar uma tabela e ler colunas/índices. 4. Selecionar procedure ou trigger e abrir o modal. 5. Usar `Investigar` ou `Consultar`.
- **Resultado esperado:** a seleção carrega a estrutura real; a procedure abre parâmetros e corpo; a trigger mostra evento, momento, ações e corpo quando disponível; `Consultar` pré-prepara o Criador. A tela é somente leitura.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a leitura depende de permissões de metadados no SQL Server.
- **Rotas/endpoints/tabelas:** `/database/explorador`; `GET /api/v1/database/tables`, `/tables/{schema}/{nome}/columns`, `/tables/{schema}/{nome}/indexes`, `/procedures`, `/procedures/{schema}/{nome}`, `/triggers`, `/triggers/{schema}/{nome}`; `sys.tables`, `sys.columns`, `sys.indexes`, `sys.procedures` e `sys.triggers`, somente leitura.
- **Status/automação:** **cenário documentado**; não há teste automatizado do Explorador.

### DB-03 — Consultar o detalhe de uma tabela

- **Objetivo:** Analisar colunas, índices, FKs, triggers, procedures, dependências e gerar um SELECT de consulta.
- **Pré-condições:** operador autenticado; tabela existente no banco conectado.
- **Passos:** 1. Abrir `/database/tabela/:schema/:tabela`. 2. Navegar pelas abas. 3. Ativar inferidas, abrir procedure/trigger ou dependência. 4. Selecionar campos/filtros e usar `Abrir no Criador` ou `Copiar SQL`.
- **Resultado esperado:** a tela mostra metadados, relacionamentos confirmados/inferidos, procedures, triggers e dependências; a navegação para tabela relacionada funciona quando o nome é identificável. O SQL de consulta rápida é apenas SELECT local e não é executado pelo sistema.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a inferência de relacionamentos é indicativa e não prova uma FK física.
- **Rotas/endpoints/tabelas:** `/database/tabela/:schema/:tabela`; `GET /api/v1/database/tables/{schema}/{nome}`, `/columns`, `/indexes`, `/relationships`, `/triggers`, `/procedures`, `/procedures/{schema}/{nome}/analysis`, `/tables/{schema}/{nome}/dependencies`; metadados `sys.*`, sem escrita.
- **Status/automação:** **cenário documentado**; não há teste automatizado do detalhe.

### DB-04 — Explorar relacionamentos

- **Objetivo:** Mapear vínculos confirmados ou inferidos de uma tabela e localizar uso de uma coluna.
- **Pré-condições:** operador autenticado; lista de tabelas carregada.
- **Passos:** 1. Abrir `/database/relacionamentos`. 2. Selecionar uma tabela. 3. Alternar Todas/Apenas confirmadas/Apenas possíveis. 4. Informar uma coluna e clicar em Buscar. 5. Clicar em origem ou destino.
- **Resultado esperado:** a tabela escolhida limita a lista; cada linha mostra tipo, origem, destino, score e motivos; a busca por coluna substitui a listagem pelo resultado de uso. A navegação para uma tabela só ocorre quando o nome contém schema e tabela. O filtro “Apenas possíveis” deve mostrar somente relações do tipo `Possivel`.
- **Estado atual/bloqueio:** limitação conhecida: no filtro “Apenas possíveis”, o frontend compara `tipo.toLowerCase()` (`possivel`) com `filtro.slice(0, -1)` (`possivei`); a comparação falha e o filtro pode retornar lista vazia mesmo com relações possíveis.
- **Rotas/endpoints/tabelas:** `/database/relacionamentos`; `GET /api/v1/database/tables`, `GET /api/v1/database/relationships?tabela=...&incluirPossiveis=...`, `GET /api/v1/database/column-usage?coluna=...`; `sys.foreign_keys` e inferências do serviço, somente leitura.
- **Status/automação:** **cenário documentado**; não há teste automatizado de relacionamentos.

### DB-05 — Montar e copiar uma consulta

- **Objetivo:** Construir um SELECT por etapas, deixar o SQL pronto para uso externo e não executar a consulta no sistema.
- **Pré-condições:** operador autenticado; tabelas e relacionamentos disponíveis.
- **Passos:** 1. Abrir `/database/consultas` ou `/database/query-builder`. 2. Escolher tabela principal, campos e tabelas relacionadas. 3. Adicionar filtros, ordenação, agrupamento/HAVING e limite. 4. Ver o SQL e copiar.
- **Resultado esperado:** o assistente percorre Tabela, Campos, Relacionamentos, Filtros, Ordenação, Resumo e SQL; a API monta o SQL e há montagem local alternativa se o serviço falhar; a tela não possui Executar nem chama `POST /query`. Parâmetros de URL e `sessionStorage` podem pré-preencher tabelas.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a montagem local é alternativa de apresentação e não representa validação SQL pelo servidor.
- **Rotas/endpoints/tabelas:** `/database/consultas` e redirecionamento `/database/query-builder`; `GET /api/v1/database/tables`, `/relationships`, `/graph`; `POST /api/v1/database/query-builder-advanced` com `{ Tabelas, Colunas, Relacionamentos, WhereConditions?, OrderBy?, GroupBy?, Limite?, Having?, Ctes? }`; nenhuma tabela de negócio é escrita.
- **Status/automação:** **cenário documentado**; não há teste automatizado do Criador de Consultas.

### DB-06 — Comparar schemas e sincronizar

- **Objetivo:** Comparar a estrutura do banco conectado com um arquivo JSON (tabela, banco inteiro ou corpos de procedures).
- **Pré-condições:** operador autenticado; arquivo `.json` **gerado pelo script de exportação da própria tela** (pode ser enviado cru, sem edição manual — o parser aceita caracteres de controle); para modo tabela, selecionar a tabela JCA; para o modo procedures, exportar as procedures com o script fornecido pela própria tela (1 linha por procedure).
- **Passos:** 1. Abrir `/database/diferencas`. 2. Escolher `Tabela única`, `Banco inteiro` ou `Procedures`. 3. Selecionar o JSON (ou o export das procedures), copiar/usar o script de exportação se necessário e clicar em `Comparar`. 4. Navegar pelas 5 abas do relatório (Tabelas faltantes, Colunas faltantes, Tamanhos divergentes, Índices divergentes ou faltantes, Chaves FK e PK divergentes ou faltantes) e conferir os cards fixos de compatíveis e %; no modo `Banco inteiro`, usar os chips `só no arquivo` e `só no banco` (sempre visíveis, em qualquer aba) para restringir listagens e badges àquele lado da diferença (clique de novo limpa); o modo `Tabela única` não tem filtro de lado; exportar CSV. 5. Gerar os scripts de correção, revisar por aba, validar (estático), copiar ou exportar `.sql`. 6. No modo procedures, conferir os corpos lado a lado, filtrar por status/busca e copiar. 7. Limpar.
- **Resultado esperado:** o relatório tem **5 abas** com badge de contagem (a primeira com contagem > 0 é a inicial; `Tabelas faltantes` só aparece no banco inteiro) e **2 cards fixos sem filtro**: compatíveis (apenas contagem) e match (%). Os botões de aba têm rótulo em destaque (fonte maior). **Cada registro da lista exibe um contador sequencial por aba (1, 2, 3…)**, com cor por severidade, e o CSV traz o mesmo número na coluna `numero`. A aba `Tamanhos divergentes` agrupa diferenças de tipo/tamanho e de nullable; `Chaves FK e PK` inclui a comparação de chave primária, que passou a ser exportada pelo script (`"pk": {"nome","colunas"}` ou `null`) e pelo backend (`SchemaInfoDto.Pk`, categoria `Pk`); JSON antigo sem `pk` pula a comparação de PK. No banco inteiro os itens são listados de forma achatada com tag da tabela e os chips clicáveis `só no arquivo` e `só no banco` aparecem **em qualquer aba** (filtram a listagem pelo lado da diferença; clicar de novo volta a `todos`) — as pills `Todos / Só no arquivo / Só no banco / Divergentes` foram removidas e o modo `Tabela única` ficou sem filtro de lado; os chips `ok` e `com diferenças` também foram removidos. Itens compatíveis (`Ok`) ficam só nos cards de contagem e no %, fora das listagens e do CSV. A geração segue o **banco JCA como fonte da verdade**: divergências e itens só no JCA não geram script; só criações vindas do arquivo (`CREATE_TABLE`, `ADD_COLUMN`, `CREATE_INDEX`, `ALTER_FK`) são geradas, cada uma com `tabela` e `severidadeOrigem`; o `CREATE TABLE` prefere a PK informada no arquivo (com fallback para índice `PK*`). No modo procedures, a resposta traz chips de status (`Compativel`, `Divergente`, `SomenteBanco`, `SomenteArquivo`), busca e comparação de corpos lado a lado, **sem** gerar SQL. A UI aceita somente JSON, com limite de 5 MB no modo tabela e 20 MB no modo banco e nas procedures. Nenhum SQL é executado e nada é persistido; a exportação CSV é local, exclui itens `Ok` e traz as colunas `aba` e `numero` (numeração por aba igual à da tela) nos dois modos de schema; a comparação não altera o banco.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a comparação depende de a conexão permitir metadados. No modo procedures, cópia pelo SSMS pode truncar corpos muito grandes (~43 mil caracteres). O parse de bulk **não ignora** linhas JSON inválidas: desde o commit `bf20b65` (2026-10-06) linha que parece JSON mas não parseia vira erro visível com o número da linha (HTTP 400), em vez de perda silenciosa que ampliava falsos `SomenteBanco`. **Correção posterior ao `bf20b65`:** o parser reparseia com escape `\u00XX` quando encontra caracteres de controle crus (0x00-0x1F) em string JSON (causa do 400 “Linha 121” no arquivo real com `0x1F` no nome) e o script de exportação do sistema passou a emitir JSON válido escapando esses controles; o nome com o caractere é **preservado** e aparece como divergência visível — o arquivo gerado pelo script pode ser enviado **cru, sem edição manual**.
- **Rotas/endpoints/tabelas:** `/database/diferencas`; `GET /api/v1/database/tables`; `POST /api/v1/database/compare-schemas` multipart com `schema`, `tabela` e `arquivo`; `POST /api/v1/database/compare-schemas-bulk` e `POST /api/v1/database/compare-procedures` multipart com `arquivo`; `POST /api/v1/database/generate-correction-scripts`, `POST /api/v1/database/generate-correction-scripts-bulk` e `POST /api/v1/database/validate-script` com JSON; leitura dos metadados `sys.*` (incluindo `sys.sql_modules` para as procedures), sem escrita.
- **Status/automação:** **cenário documentado**; não há teste automatizado da Sincronização.

### DB-07 — Verificar a configuração da conexão

- **Objetivo:** Conferir a configuração efetiva e testar a conectividade sem expor credenciais.
- **Pré-condições:** operador autenticado; backend com configuração de conexão disponível.
- **Passos:** 1. Abrir `/database/configuracao`. 2. Ler servidor, porta, banco, usuário e indicadores de segurança. 3. Clicar em `Testar conexão`. 4. Ler o resultado e a duração.
- **Resultado esperado:** a tela é somente leitura; a senha aparece mascarada e a configuração é gerenciada fora da interface. O teste chama `GET /status`; não há `PUT /config` nem `POST /test-connection` no código atual.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a máscara é apresentação e a proteção efetiva depende de não expor logs, respostas ou configuração do ambiente.
- **Rotas/endpoints/tabelas:** `/database/configuracao`; `GET /api/v1/database/config` e `GET /api/v1/database/status`, ambos `[Authorize]`; nenhuma tabela da Central e nenhum segredo deve ser copiado para o resultado.
- **Status/automação:** **cenário documentado**; não há teste automatizado da Configuração.

