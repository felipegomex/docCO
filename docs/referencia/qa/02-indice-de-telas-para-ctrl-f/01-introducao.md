---
id: 02-indice-de-telas-para-ctrl-f.01
titulo: "Introdução"
tipo: referencia
area: qa
sistema: central
tags: [qa, telas, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - docs/referencia/qa
testes:
  - docs/
tokens_estimados: 3474
ficha: docs/modulos/README.md
---
> Parte de [Índice de telas para `Ctrl+F`](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### Introdução

> Parte de [docs/referencia/qa-historias.md](../../qa-historias.md).

| Grupo | Tela ou fluxo | Rota ativa | História | Status |
|---|---|---|---|---|
| Login/Base | Login | `/login` | LB-01 | cenário documentado |
| Login/Base | Renovação, expiração e logout | rotas protegidas | LB-02 | cenário documentado |
| Login/Base | Home | `/` | LB-03 | cenário documentado |
| Login/Base | Busca global do Header e da Home | `/` e Header | LB-04 | validado em teste |
| Login/Base | Layout, menu, breadcrumb e escala | todas as rotas autenticadas | LB-05 | cenário documentado |
| Conhecimento/Ferramentas | Conhecimento e Resolver (aba Base de Conhecimento) | `/base-conhecimento?aba=conhecimento&trilha=resolver` | CF-01 | cenário documentado |
| Conhecimento/Ferramentas | Guias SQL, Rede e Infra (abas da Base de Conhecimento) | `/base-conhecimento?aba=conhecimento` | CF-02 | cenário documentado |
| Conhecimento/Ferramentas | Ferramentas e detalhe | `/ferramentas`, `/ferramentas/detalhe/:id` | CF-03 | cenário documentado |
| Acessos Empresas | Lista de unidades (filtros e paginação) | `/acessos/clientes` | CF-04 | cenário documentado |
| Acessos Empresas | Assistente de nova unidade (3 etapas, POST agregado) | `/acessos/clientes/novo` | CF-04 | cenário documentado |
| Acessos Empresas | Detalhe, gate de senha, abas e revelação | `/acessos/clientes/:id` | CF-12 | cenário documentado |
| Conhecimento/Ferramentas | FAQ/Glossário (aba da Base de Conhecimento) | `/base-conhecimento?aba=glossario` | CF-05 | cenário documentado |
| Conhecimento/Ferramentas | Cursos e detalhe | `/cursos`, `/cursos/detalhe/:id` | CF-06 | cenário documentado |
| Conhecimento/Ferramentas | Fraseologias | `/fraseologia` | CF-07 | cenário documentado |
| Conhecimento/Ferramentas | Modelo de chamados | `/modelo-chamados` | CF-08 | cenário documentado |
| Conhecimento/Ferramentas | Stack e Políticas | `/base-conhecimento?aba=stack`, `/politica` | CF-09 | cenário documentado |
| Conhecimento/Ferramentas | Visão ADM | `/visao-adm` | CF-10 | cenário documentado |
| Conhecimento/Ferramentas | Detalhe de procedimento | `/visao-adm/detalhe/:id` | CF-11 | cenário documentado |
| Conhecimento/Ferramentas | FAQ Sistema — catálogo de telas | `/faq-sistema` | CF-13 | cenário documentado |
| Implantação | Dashboard | `/implantacao/dashboard` | IM-01 | cenário documentado |
| Implantação | Projetos | `/implantacao/projetos` | IM-02 | cenário documentado |
| Implantação | Novo/editar Projeto | `/implantacao/projetos/novo`, `/implantacao/projetos/:id/editar` | IM-03 | cenário documentado |
| Implantação | Detalhe do Projeto e etapas | `/implantacao/projetos/:id` | IM-04 | cenário documentado |
| Implantação | Lista de Tarefas | `/implantacao/tarefas` | IM-05 | cenário documentado |
| Implantação | Nova/editar Tarefa | `/implantacao/tarefas/novo`, `/implantacao/tarefas/:id/editar` | IM-06 | validado em teste |
| Implantação | Kanban: filtros, criação e movimento (modo legado) | `/kanban` | IM-07 | cenário documentado |
| Implantação | Kanban: drawer, menu e lote | `/kanban` | IM-08 | cenário documentado |
| Kanban Geral | Quadro por setor (tarefas, projetos e ambos) | `/kanban` | KG-01 | validado em teste |
| Kanban Geral | Gargalos: aging e fechar bloqueio | `/kanban/gargalos` | KG-02 | validado em teste |
| Kanban Geral | Administração do quadro (catálogo de colunas, visão do setor, funções, etapas) | `/kanban/admin/colunas` | KG-03 | cenário documentado |
| Navegação/RBAC | Sidebar por setor e Ferramentas compartilhadas | todas as rotas autenticadas | NAV-01 | cenário documentado |
| Navegação/RBAC | Visão Geral por setor (Suporte, Desenvolvimento e CIAA) | `/suporte/visao-geral`, `/desenvolvimento/visao-geral`, `/ciaa/visao-geral` | NAV-02 | cenário documentado |
| Navegação/RBAC | Projetos do CIAA com `funcaoId=5` | `/ciaa/projetos`, `/ciaa/projetos/novo`, `/ciaa/projetos/:id`, `/ciaa/projetos/:id/editar` | NAV-03 | cenário documentado |
| Navegação/RBAC | Base de Conhecimento única e redirects | `/base-conhecimento` | NAV-04 | cenário documentado |
| Navegação/RBAC | Ideias e Melhorias reutilizado por Sistema/CIAA/Implantação | `/ideias/lista` | NAV-05 | cenário documentado |
| Auditoria | Trilhas de acessos e implantação (filtros e paginação) | `/auditoria` | AU-01 | cenário documentado |
| Sistema (RBAC) | Entrar em Funções Administrativas e enxergar o menu | `/sistema/funcoes-administrativas` | RB-01 | cenário documentado |
| Sistema (RBAC) | Consultar as próprias permissões e o guard de rota | `/sistema/operadores` | RB-02 | cenário documentado |
| Sistema (RBAC) | Master cria operador e define cargo/setor | `/sistema/operadores` | RB-03 | cenário documentado |
| Sistema (RBAC) | Tech Lead restrito ao próprio setor e aos cargos abaixo | `/sistema/operadores` | RB-04 | cenário documentado |
| Sistema (RBAC) | Inativar, trocar senha e recusar exclusão | `/sistema/operadores` | RB-05 | cenário documentado |
| Sistema (RBAC) | Matriz de permissões: Master, Tech Lead e negação | `/sistema/permissoes` | RB-06 | validado em teste |
| Database | Shell e Visão Geral | `/database`, `/database/visao-geral` | DB-01 | cenário documentado |
| Database | Explorador | `/database/explorador` | DB-02 | cenário documentado |
| Database | Detalhe de tabela | `/database/tabela/:schema/:tabela` | DB-03 | cenário documentado |
| Database | Relacionamentos | `/database/relacionamentos` | DB-04 | cenário documentado |
| Database | Criador de Consultas | `/database/consultas`, `/database/query-builder` | DB-05 | cenário documentado |
| Database | Diferenças/Sincronização | `/database/diferencas` | DB-06 | cenário documentado |
| Database | Configuração | `/database/configuracao` | DB-07 | cenário documentado |
| Empresa/Agenda | Empresa e Onboarding inicial | `/empresa`, `/empresa/onboarding` | EA-01 | cenário documentado |
| Empresa/Agenda | Capítulo de Onboarding | `/empresa/onboarding/capitulo/:id` | EA-02 | cenário documentado |
| Empresa/Agenda | Navegação e filtros da Agenda | `/agenda` | EA-03 | cenário documentado |
| Empresa/Agenda | Formulário e recorrência de evento | `/agenda` + modal | EA-04 | cenário documentado |
| Empresa/Agenda | Permissões, conflitos e persistência | `/agenda` + API | EA-05 | cenário documentado |
| Início | Semana de tarefas do operador | `/` | PL-01 | cenário documentado |
| Dashboard | Home com KPIs | `/dashboard` | DBN-01 | cenário documentado |
| Dashboard | Kanban de Leads | `/dashboard/comercial` | DBN-02 | cenário documentado |
| Dashboard | Marcar "cliente viu o contato" | `/dashboard/comercial` (toggle no card / checkbox no modal) | DBN-07 | cenário documentado |
| Dashboard | Criar oportunidade e operar Propostas | `/dashboard/comercial` (modal do lead) e `/dashboard/comercial/propostas` | DBN-03 | cenário documentado |
| Dashboard | Base de clientes (com grupos e filtros) | `/dashboard/administrativo` | DBN-04 | cenário documentado |
| Dashboard | Financeiro | `/dashboard/financeiro` | DBN-05 | cenário documentado |
| Dashboard | Histórico de contatos do lead (prospecção) | `/dashboard/comercial` (modal/timeline) | DBN-06 | cenário documentado |
| Admin/Kanban F | Tipos de Projeto | `/admin/cadastros/tipos-projeto`, `/novo`, `/editar/:id` | AK-01 | cenário documentado |
| Admin/Kanban F | Kanban F (atalho do menu) | `/kanban` (sem `?perfil=F` desde 2026-10-01; `?perfil=` segue como deep link) | AK-02 | cenário documentado |
| Admin/Kanban F | Entradas administrativas | `/admin`, `/admin/cadastros/tipos-projeto` | AK-03 | cenário documentado |
| Chat/Conhecimento | Perguntar ao JOTA (agente por setor) e ver fontes | `/chat` e widget global no `MainLayout` | CJ-01 | cenário documentado |
| Chat/Conhecimento | Navegação e revisão de documentos | `/conhecimento` e filhos | CJ-02 | cenário documentado |
| Ideias, Melhorias e Erros | Listagem, filtros e paginação | `/ideias/lista` | ID-01 | cenário documentado (módulo não publicado) |
| Ideias, Melhorias e Erros | Nova solicitação (3 tipos) | `/ideias/nova`, `/ideias/nova/:tipo` | ID-02 | cenário documentado (módulo não publicado) |
| Ideias, Melhorias e Erros | Detalhe, timeline e ações por fase | `/ideias/:id` | ID-03 | cenário documentado (módulo não publicado) |
| Ideias, Melhorias e Erros | Gestão de gestores | /gestores-ideias | ID-04 | cenário documentado (módulo não publicado) |
| Ideias, Melhorias e Erros | Excluir solicitação (administrador ou gestor) | `/ideias/lista`, `/ideias/:id` | ID-05 | cenário documentado (módulo não publicado) |
| Ideias, Melhorias e Erros | Leitor do setor CIAA (somente leitura) | `/ideias/lista`, `/ideias/:id` | ID-06 | validado em teste |
| Atendimento WhatsApp | Receber mensagem e abrir o atendimento na fila | `/whatsapp` | WA-01 | cenário documentado (exercitado no piloto da HML em 2026-10-21) |
| Atendimento WhatsApp | Assumir o atendimento pela primeira resposta | `/whatsapp/atendimento/:id` | WA-02 | cenário documentado (exercitado no piloto da HML em 2026-10-21) |
| Atendimento WhatsApp | Transferir o atendimento com histórico | `/whatsapp/atendimento/:id` | WA-03 | cenário documentado (piloto em operação na HML em 2026-10-21) |
| Atendimento WhatsApp | Devolver o atendimento à fila | `/whatsapp/atendimento/:id` | WA-04 | cenário documentado (piloto em operação na HML em 2026-10-21) |
| Atendimento WhatsApp | Resolver com mensagem de encerramento | `/whatsapp/atendimento/:id` | WA-05 | cenário documentado (exercitado no piloto da HML em 2026-10-21) |
| Atendimento WhatsApp | Resolver sem enviar mensagem | `/whatsapp/atendimento/:id`, `/whatsapp` | WA-05b | cenário documentado |
| Atendimento WhatsApp | Reabrir o atendimento | `/whatsapp/atendimento/:id` | WA-06 | cenário documentado (exercitado no piloto da HML em 2026-10-21) |
| Atendimento WhatsApp | Aguardar o cliente e retomar | `/whatsapp/atendimento/:id` | WA-07 | cenário documentado (piloto em operação na HML em 2026-10-21) |
| Atendimento WhatsApp | Registrar nota interna | `/whatsapp/atendimento/:id` | WA-08 | cenário documentado (piloto em operação na HML em 2026-10-21) |
| Atendimento WhatsApp | Administrar os números conectados | `/whatsapp/numeros` | WA-09 | cenário documentado (piloto em operação na HML em 2026-10-21) |
| Atendimento WhatsApp | Consultar os indicadores do atendimento | `/whatsapp/metricas` | WA-10 | cenário documentado (piloto em operação na HML em 2026-10-21) |
| Atendimento WhatsApp | Cadastrar o número e obter o QR sem erro | `/whatsapp/numeros` | WA-11 | cenário documentado (defeitos do piloto de 2026-10-21) |
| Atendimento WhatsApp | Listar os números sem `500` | `/whatsapp/numeros` | WA-12 | cenário documentado (defeitos do piloto de 2026-10-21) |
| Atendimento WhatsApp | Responder mensagem com o `BaseUrl` sem barra final | `/whatsapp/atendimento/:id` | WA-13 | cenário documentado (defeitos do piloto de 2026-10-21) |
| Atendimento WhatsApp | Remover o número e conferir que a sessão sumiu do gateway | `/whatsapp/numeros` | WA-14 | cenário documentado (defeitos do piloto de 2026-10-21) |
| Atendimento WhatsApp | Publicar o backend e confirmar que as rotas respondem | servidor IIS (backend) | WA-15 | cenário documentado (defeitos do piloto de 2026-10-21) |
| Atendimento WhatsApp | Gravar o status canônico da sessão sem estourar a coluna | `/whatsapp/numeros` | WA-16 | validado em teste (defeito do piloto de 2026-10-21) |
| Atendimento WhatsApp | Recuperar a linha da fila presa em "processando" | servidor IIS (backend) | WA-17 | validado em teste (defeito do piloto de 2026-10-21) |
| Atendimento WhatsApp | Consultar a saúde do canal com a sessão ativa | `/whatsapp/numeros` | WA-18 | cenário documentado (defeito do piloto de 2026-10-21) |
| Atendimento WhatsApp | Ver a tela de números refletir o status sozinha | `/whatsapp/numeros` | WA-19 | cenário documentado (defeito do piloto de 2026-10-21) |
| Atendimento WhatsApp | Filtrar a fila por "sem responsável" | `/whatsapp` | WA-20 | cenário documentado |
| Atendimento WhatsApp | Acessar "Minhas Conversas" e abrir uma conversa | `/whatsapp/minhas-conversas` | WA-21 | cenário documentado |
| Atendimento WhatsApp | Consultar o histórico do cliente (aba lateral "Cliente") | `/whatsapp/atendimento/:id` | WA-22 | cenário documentado |
| Atendimento WhatsApp | Visualizar mídia da mensagem (imagem, áudio, vídeo, documento) | `/whatsapp/atendimento/:id` | WA-23 | cenário documentado |
| Atendimento WhatsApp | Visualizar o layout do Assistente IA (aba lateral "Assistente IA") | `/whatsapp/atendimento/:id` | WA-24 | cenário documentado |
| Atendimento WhatsApp | Consultar métricas de qualidade (taxa reabertura, FCR, transferências, CSAT) | `/whatsapp/metricas` (aba Qualidade) | WA-30 | cenário documentado |
| Atendimento WhatsApp | Consultar métricas por cliente (volume, tendência) | `/whatsapp/metricas` (aba Clientes) | WA-32 | cenário documentado |
| Atendimento WhatsApp | Consultar auditoria do canal (filtros usuário/ação/objeto/período) | `/whatsapp/auditoria` | WA-33 | cenário documentado (gravação real nas escritas) |
| Atendimento WhatsApp | Configurar IA do canal (provedor, chave API, modelo, timeout) | `/whatsapp/ia-configuracao` | WA-34 | cenário documentado |
| Atendimento WhatsApp | Usar Assistente IA real (resumo, classificação, sugestões) | `/whatsapp/atendimento/:id` (aba Assistente IA) | WA-35 | cenário documentado (sem crash/recursão) |
| Atendimento WhatsApp | Kanban com 4 colunas por padrão e filtro "incluir arquivados" revelando Finalizado/Arquivado/Visto | `/whatsapp` | WA-36 | cenário documentado |
