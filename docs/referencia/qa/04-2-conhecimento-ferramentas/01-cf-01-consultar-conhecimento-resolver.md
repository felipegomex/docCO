---
id: 04-2-conhecimento-ferramentas.01
titulo: "CF-01 — Consultar Conhecimento/Resolver (+6)"
tipo: referencia
area: qa
sistema: central
tags: [qa, conhecimento, ferramentas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features/conhecimento
  - backend/Central_BackEnd/Controllers
testes:
  - docs/
tokens_estimados: 3457
ficha: docs/modulos/chat-conhecimento.md
---
> Parte de [2. Conhecimento/Ferramentas](./INDEX.md). Ficha: [Chat e Conhecimento](../../../modulos/chat-conhecimento.md).

### CF-01 — Consultar Conhecimento/Resolver

- **Objetivo:** Encontrar o procedimento de triagem e a seção adequada para um incidente.
- **Pré-condições:** operador autenticado; conteúdo estático do guia disponível.
- **Passos:** 1. Abrir `/base-conhecimento?aba=conhecimento&trilha=resolver` (ou o antigo `/trilhas/resolver`, que redireciona). 2. Informar um termo no filtro. 3. Abrir uma seção do índice ou um exemplo rápido. 4. Limpar o filtro e abrir outra seção.
- **Resultado esperado:** o guia tem oito seções; o filtro reduz o índice, o estado vazio orienta outro termo e os exemplos abrem/rolam para “Exemplos Práticos”. O módulo editorial separado fica em `/conhecimento`.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; o guia é estático e não depende de persistência de backend.
- **Rotas/endpoints/tabelas:** `/base-conhecimento` (aba Conhecimento Rápido, trilha Resolver) e `/trilhas/resolver` (redirect); nenhum endpoint e nenhuma tabela; conteúdo em `frontend/src/app/features/trilhas/trilhas.component.ts` e seu template.
- **Status/automação:** **cenário documentado**; não há teste do guia.
### CF-02 — Consultar as trilhas SQL, Rede e Infra

- **Objetivo:** Oferecer referências operacionais estáticas e comandos copiáveis para investigação.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/base-conhecimento?aba=conhecimento&trilha=sql` e pesquisar na biblioteca/categoria. 2. Copiar uma consulta. 3. Alternar para Rede e expandir seções. 4. Alternar para Infra e expandir seções.
- **Resultado esperado:** SQL mantém o acordeão de Orientação, filtra a biblioteca e informa cópia; Rede e Infra exibem seus guias estáticos. O sistema não executa SQL nem altera o banco. Os antigos `/trilhas/sql`, `/trilhas/rede` e `/trilhas/infra` redirecionam para as sub-abas correspondentes.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; os comandos são copiados para uso externo, não executados pelo sistema.
- **Rotas/endpoints/tabelas:** `/base-conhecimento` (aba Conhecimento Rápido) e `/trilhas/sql|rede|infra` (redirects); nenhum endpoint/tabela; dados em `biblioteca-sql.data.ts` e nos componentes estáticos.
- **Status/automação:** **cenário documentado**; não há testes dessas telas.
### CF-03 — Filtrar e abrir Ferramentas

- **Objetivo:** Localizar uma ferramenta de suporte e abri-la — dentro da Central quando o site permite, ou em nova aba quando não permite.
- **Pré-condições:** operador autenticado; `ferramentas.data.ts` carregado.
- **Passos:** 1. Abrir `/ferramentas`. 2. Pesquisar por nome/descrição e selecionar `Todas` ou uma categoria. 3. Ativar/desativar a pill **"Disponíveis aqui"** para ver só as ferramentas que abrem dentro da Central. 4. Limpar filtros. 5. Em um card com badge **"Abre aqui"** clicar **"Usar aqui"** e conferir o workspace integrado (header/sidebar visíveis; toolbar com recarregar, comparar, trocar, tela cheia interna, nova aba e fechar). 6. No workspace, testar Expandir (oculta header/sidebar; `Esc` ou Sair restaura sem recriar o iframe), Comparar (2º painel com seletor e busca), divisor arrastável, recarga e troca independentes por painel, fechar um painel (o outro assume) e voltar à Central. 7. Em um card com badge **"Site externo"** clicar **"Abrir site"** e confirmar a nova aba com o toast de aviso. 8. Abrir o detalhe de uma ferramenta e repetir "Usar aqui"/"Nova aba"; testar um ID inexistente e um `?painel=` com id inválido (estado vazio com retorno).
- **Resultado esperado:** o contador mostra "N ferramentas encontradas · M abrem aqui" e a legenda explica os dois modos; o card com `embutivel: true` navega para `/ferramentas/workspace?painel=:id`, que exibe o `FerramentaEmbedComponent` como painel integrado ao layout (sem modal); o botão Comparar abre o segundo painel (`?comparar=:id`) lado a lado com estado independente; a tela cheia interna expande sem perder a navegação do site; sites que bloqueiam iframe exibem banner com alternativa em nova aba; o com `embutivel: false` abre `window.open` e exibe o toast "Este site não permite exibição interna — aberto em nova aba."; o detalhe mostra o mesmo badge e a observação de que o site sempre abre em nova aba; uma referência local inexistente mostra estado “Ferramenta não encontrada” com retorno à Central.
- **Estado atual/bloqueio:** a fonte das ferramentas é local e não há persistência de backend; a lista `embutivel` é manual porque o bloqueio (`X-Frame-Options`/`frame-ancestors`) não é detectável em runtime — 6 de 24 ferramentas abrem em painel (5 confirmadas + Dillinger com compatibilidade desconhecida e fallback para nova aba); 16 são downloads e MarkItDown é só documentação. A comparação permite a mesma ferramenta nos dois painéis (instâncias independentes por slot); downloads/docs nunca viram painel, por allowlist. Um site pode mudar esse comportamento sem aviso.
- **Rotas/endpoints/tabelas:** `/ferramentas`, `/ferramentas/detalhe/:id`, `/ferramentas/workspace?painel=:id&comparar=:id`; nenhum endpoint/tabela; `FerramentasService`, `FerramentaWorkspaceService`, `ferramentas.data.ts` e `DashboardToastService` são locais.
- **Status/automação:** **cenário documentado**; não há teste da lista ou do detalhe.
### CF-04 — Consultar a lista de unidades de Acessos

> CF-04 e CF-12 formam o grupo **Acessos Empresas**; a seção 9 traz o cenário do detalhe. Antes da reescrita, esta história cobria a tela `/ferramentas/acessos` que lia uma planilha do Google Sheets (ver "Regressão/histórico").

- **Objetivo:** Localizar a unidade (cliente/filial) que se quer consultar, com filtros e paginação, e ver o estado real da proteção das credenciais — incluindo criar uma unidade **com acessos de uma vez só** pelo assistente.
- **Pré-condições:** operador autenticado; grupos cadastrados para o filtro (opcional); para criar via ERP, empresas ativas em `tbcliente`.
- **Passos:** 1. Abrir `/acessos/clientes` (grade de cards — `ListaClientesCardsComponent`; a rota `/clientes/tabela` abre a lista paginada `ListaClientesComponent`). 2. Observar o breadcrumb manual (`Início > Acessos das Empresas`), a **linha de topo única** (total de empresas + único botão `+ Novo Acesso`) e o badge de status (● Ativo/● Inativo) de cada card. 3. Na tabela, pesquisar por fantasia/CNPJ e clicar **Buscar** (ou `Enter` — `lista-clientes.component.html`/`.acesso-busca`); nos cards a busca filtra ao digitar (debounce de 300 ms). Escolher o grupo e alternar "somente ativos". 4. Paginar. 5. Clicar no `+ Novo Acesso` ao lado do total — abre o **assistente** em `/acessos/clientes/novo` (etapa 1 *Empresa*: no typeahead, digitar ≥ 2 caracteres de uma empresa do ERP e **selecionar** uma sugestão — a seleção é **obrigatória**, não existe cadastro manual de unidade; os campos são só preenchidos, nada é criado ainda). 6. Avançar para a etapa 2 *Tipos de acesso* e **clicar no card do tipo** para marcá-lo (um clique marca/desmarca; o "Próximo" só habilita com ao menos um tipo). 7. Na etapa 3 *Configurar*, preencher cada tipo nos cards (form em modo coleta) e concluir — o assistente pede a senha (ticket de criação) e envia **um único** `POST /clientes` com a empresa + todos os acessos. 8. Usar o **menu ⋮** de um card: Visualizar acessos, Gerenciar acessos, Editar unidade (modal `FormClienteComponent`) ou Inativar unidade (com gate). 9. Confirmar que cada card tem `min-height: 14rem` e o estilo `.acesso-breadcrumb` está aplicado.
- **Resultado esperado:** a grade (`/acessos/clientes`) exibe cards com `min-height: 14rem`, cada um mostrando fantasia, grupo, código ERP (`#N`), os **4 badges de recursos** (RDP, VPN, AnyDesk, Actyon Web — BANCO fora da grade) e o **badge de status**; o breadcrumb manual (`Início > Acessos das Empresas`) e a **linha de topo única** (total de empresas + `+ Novo Acesso`, único ponto de entrada do assistente — ausente do header e do estado vazio) estão presentes no topo; a lista paginada (`/clientes/tabela`) mostra fantasia, CNPJ mascarado, grupo, as 5 flags de acesso e a data de cadastro; a paginação respeita `page`/`pageSize` e o total. O aviso da cifra aparece no topo enquanto as senhas estiverem em texto puro. No assistente, o typeahead só busca com 2+ caracteres e só lista empresas **ativas** e **não cria nada** na etapa 1 (sem seleção o "Próximo" fica desabilitado); na etapa 2, **clicar no card do tipo marca/desmarca** (o Próximo exige ao menos um tipo). A gravação acontece só no final, transacionalmente. Se a empresa **já tinha** unidade (mesmo `CLIENTE_ID`), o backend devolve `jaExistia: true` e a tela mostra `toast.info` "Esta empresa já tinha uma unidade" **abrindo a unidade existente** — nunca duplica (a chave é o vínculo com o ERP, não o CNPJ). Depois de concluir, o navegador vai para `/acessos/clientes/{id}`. Inativar pede confirmação **informando que os acessos também serão inativados**, com ticket `EDITAR` da **unidade** (sem ele o `DELETE` cai em 403).
- **Estado atual/bloqueio:** sem limitação específica desta tela. A inativação é **soft delete em cascata** (cliente + os 5 tipos de acesso) e nada é apagado fisicamente. **No máximo uma unidade por empresa do ERP** (`CAO_ClienteId` com índice único filtrado — a chave de unicidade é o `CLIENTE_ID`); **identidade não é gravada na `tbclienteacesso`** (`CAO_Fantasia`/`CAO_Cnpj` foram dropadas): fantasia, CNPJ e grupo exibidos vêm da `tbcliente` por JOIN, e corrigem-se **no ERP**. Grupos vêm de `tbcliente_grupo` (somente leitura, sem CRUD). A rota `GET /clientes/busca-erp` é literal e responde antes de `clientes/{id:int}`. A edição da empresa continua em **modal**, mas só altera **situação e observação** (identidade é somente leitura; o modal anexa `X-Acesso-Ticket` via `headers = headers.set(...)`, pois `HttpHeaders` é imutável); o assistente é só para criação.
- **Rotas/endpoints/tabelas:** `/acessos/clientes`, `/acessos/clientes/novo`; `GET /api/v1/acessos/clientes` (paginado), `GET /acessos/clientes/busca-erp?busca=&take=` (typeahead do ERP), `GET /acessos/grupos`, `GET /acessos/estado-cifra`, `POST /acessos/clientes` (`{ clienteId, observacao?, acessos? }` → `201` criou / `200` `jaExistia`), `PUT /acessos/clientes/{id}`, `DELETE /acessos/clientes/{id}`; `tbclienteacesso` (+ FK `CAO_GrupoId` → `tbcliente_grupo`, FK `CAO_ClienteId` → `tbcliente`), `tbcliente_grupo` (grupos), `tbcliente` (empresa do ERP), `tbacessordp`/`tbacessovpn`/`tbacessoanydesk`/`tbacessobanco`/`tbacessoactyonweb` (no INSERT agregado).
- **Status/automação:** **cenário documentado**; não há teste de UI do módulo.
### CF-05 — Navegar no FAQ/Glossário

- **Objetivo:** Consultar os capítulos do glossário interno sem sair da área logada.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/base-conhecimento?aba=glossario` (ou o antigo `/ferramentas/faq`, que redireciona). 2. Selecionar um dos sete capítulos. 3. Rolar o conteúdo e alternar entre capítulos.
- **Resultado esperado:** o capítulo selecionado é destacado e o conteúdo correspondente é renderizado; não há chamada de API nem estado de carregamento no backend.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; o teste existente é apenas de criação do componente.
- **Rotas/endpoints/tabelas:** `/base-conhecimento` (aba Glossário) e `/ferramentas/faq` (redirect); nenhum endpoint/tabela; `faq.component.ts` e `faq.component.html`.
- **Status/automação:** **cenário documentado**; `faq.component.spec.ts` contém somente um teste de fumaça de criação, insuficiente para marcar a navegação como validada.
### CF-06 — Filtrar cursos e abrir detalhe

- **Objetivo:** Encontrar cursos, organizá-los por categoria/nível e abrir o detalhe ou o player de vídeo quando disponível.
- **Pré-condições:** operador autenticado; dados locais de cursos carregados.
- **Passos:** 1. Abrir `/cursos`. 2. Pesquisar, escolher categoria e nível. 3. Alternar a aba Cursos/Trilhas. 4. Abrir um card ou `/cursos/detalhe/:id` e usar o próximo curso quando disponível.
- **Resultado esperado:** filtros e contagem refletem a seleção; o detalhe mostra categoria, descrição, trilhas e player/link quando o dado existe; ID inexistente mostra estado não encontrado.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; os cursos e seus detalhes são carregados de dados locais.
- **Rotas/endpoints/tabelas:** `/cursos`, `/cursos/detalhe/:id`; nenhum endpoint/tabela; `CursosService` e `cursos-novos.data.ts` são locais.
- **Status/automação:** **cenário documentado**; não há teste de cursos.
### CF-07 — Filtrar e copiar Fraseologias

- **Objetivo:** Localizar uma resposta de atendimento, ajustar variáveis e copiar o texto final.
- **Pré-condições:** operador autenticado; conteúdo estático disponível.
- **Passos:** 1. Abrir `/fraseologia`. 2. Pesquisar ou escolher categoria. 3. Copiar uma mensagem ou abrir edição de variáveis. 4. Informar valores e copiar.
- **Resultado esperado:** apenas mensagens compatíveis com busca/categoria permanecem visíveis; a cópia usa a área de transferência com alternativa local e exibe uma notificação de sucesso/erro. Nenhuma mensagem é gravada no backend.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a alternativa de cópia depende do suporte do navegador e do contexto seguro para a área de transferência.
- **Rotas/endpoints/tabelas:** `/fraseologia`; nenhum endpoint/tabela.
- **Status/automação:** **cenário documentado**; não há teste automatizado da Fraseologia.
