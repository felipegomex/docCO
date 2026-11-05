---
id: 11-10-fluxos-de-dados.05
titulo: "10.5 Database"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, fluxos]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 1308
ficha: docs/modulos/database-explorer.md
---
> Parte de [10. Fluxos de dados](./INDEX.md). Ficha: [Database Explorer](../../../modulos/database-explorer.md).

### 10.5 Database

`DatabaseShellComponent` carrega status e informações gerais. Páginas e componentes internos chamam `DatabaseService` para metadados, relações, busca, procedures, triggers, consultas geradas, comparação de schemas e de procedures e scripts de correção. O estado visual fica nos componentes; o serviço é a única porta HTTP do módulo.

`DbSincronizacaoComponent` executa a comparação em três modos — `Tabela única`, `Banco inteiro` e `Procedures` — e monta o resumo com **5 abas** (`ABAS_RELATORIO`/`abaDeCategoria`): `TABELAS FALTANTES` (somente no modo banco inteiro), `COLUNAS FALTANTES`, `TAMANHOS DIVERGENTES` (agrupa as categorias `Tipo` + `Nullable`), `ÍNDICES DIVERGENTES OU FALTANTES` e `CHAVES FK E PK DIVERGENTES E FALTANTES` (categorias `Fk` + `Pk`). Cada aba tem badge de contagem e a inicial é a primeira com contagem > 0 (`tabelas` no banco / `colunas` na tabela); os botões de aba usam rótulo maior (`font-size: 0.95rem`, padding ampliado). Ao lado das abas ficam **2 cards fixos, sem filtro**: `compatíveis` (apenas contagem — `oks` na tabela única, `tabelasOk` no banco) e `match` (percentual); a severidade vira só cor de item. **Cada registro listado traz um badge numerado sequencial por aba (1..N)**, com cor por severidade (`data-sev`), e o CSV usa a mesma numeração na coluna `numero` (contador por rótulo de aba, gerado em `exportarCsv()`); o antigo `numeroCritico`/`marcarNumeros` foi removido. No modo banco inteiro os itens são achatados por aba com tag da tabela e os chips `só no arquivo` e `só no banco` ficam em qualquer aba — ambos clicáveis: alternam o signal `filtroLado` entre `somenteArquivo`/`somenteBanco` e `todos` (clique de novo limpa); os chips `ok` e `com diferenças` foram removidos (ok/com diferenças seguem só nos contadores). os filtros/expanders antigos por severidade foram removidos (`filtro`, `filtroBulk`, `tabelaExpandida`). **Filtro de lado (2026-10-07, commit `a159795`)**: as pills `Todos / Só no arquivo / Só no banco / Divergentes` e o computed `contagensLado` (e os estilos `.db-sync__filtros`/`.db-sync__pill*`) foram **removidas** dos modos `Tabela única` e `Banco inteiro`; restam os 2 chips `só no arquivo`/`só no banco`, **sempre visíveis no modo banco inteiro** (antes só na aba `TABELAS FALTANTES`), que filtram as listagens e os badges de aba 100% no cliente via signal `filtroLado` (padrão `todos`) — o modo `Tabela única` ficou sem filtro de lado; a regra `ladoDaDiferenca(d)` usa `esperado` (banco JCA) × `encontrado` (arquivo): só JCA → `somenteBanco`, só arquivo → `somenteArquivo`, ambos → `divergente`; o filtro zera em `setModo`/`limparTudo`, gera mensagem própria em `vazioAba()` e **não** altera o CSV/exportação (filtro é de visualização). Itens `Ok` continuam fora das listagens e do CSV (somente contadores e %). O export CSV client-side traz as colunas `aba` e `numero` (mesmo número sequencial exibido no badge do item, contado por aba; no banco inteiro a iteração é tabela a tabela, na ordem da tela) nos dois modos de schema. O componente também gera o script de exportação JSON (schemas — agora com o campo `pk` em cada tabela, com `FOR XML PATH` compatível com SQL Server 2005+) ou T-SQL de 1 linha por procedure (procedures) para o SSMS; no modo procedures há um script próprio (`scriptSqlProcedures`, com escaping de `\`, aspas, quebras de linha e tabs) e o CSV/ação de scripts fica indisponível. A geração de correção chama `gerarScripts`/`gerarScriptsBulk` **sem** corpo de opções.

`DbScriptsCorrecaoComponent` recebe o resultado da comparação (tabela única ou bulk) com `filtroSeveridade` fixo em `null` (o painel de scripts não é mais filtrado pela aba ativa) e expõe três abas (Criação, Alterações, Índices e FKs) com badges de contagem por aba, cards por script (severidade, badge de `tabela` no modo bulk, SQL com realce próprio, copiar, validar (estático) e consulta de verificação). Os scripts são somente criações vindas do arquivo (o banco JCA é a fonte da verdade), cada um com `severidadeOrigem` (usada pelo filtro interno quando `filtroSeveridade` não é nulo — hoje a tela principal envia `null`, então nenhum filtro é aplicado). O rodapé mostra resumo/impacto e `RevisaoManual`; as ações exportam o conjunto como arquivo `.sql` (Blob) ou copiam tudo. Não há checkboxes de backup/modo estrito (as opções saíram da UI). Ele é montado dentro de `DbSincronizacaoComponent` e limpa os scripts gerados sempre que a comparação muda.

`DbProceduresResultadoComponent` recebe `ProceduresComparisonResult` no modo procedures: chips de contagem por `status` (`Compativel`, `Divergente`, `SomenteBanco`, `SomenteArquivo`), pills de filtro por status, busca por nome e a listagem com os corpos lado a lado (lado único quando só existe um dos lados), botão copiar por lado com fallback `execCommand` e export CSV (`schema,nome,status,tamanhoJca,tamanhoArquivo`). Não há geração de scripts neste modo.

