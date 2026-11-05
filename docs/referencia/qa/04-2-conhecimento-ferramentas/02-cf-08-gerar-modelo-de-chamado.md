---
id: 04-2-conhecimento-ferramentas.02
titulo: "CF-08 — Gerar Modelo de Chamado (+4)"
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
tokens_estimados: 1475
ficha: docs/modulos/chat-conhecimento.md
---
> Parte de [2. Conhecimento/Ferramentas](./INDEX.md). Ficha: [Chat e Conhecimento](../../../modulos/chat-conhecimento.md).

### CF-08 — Gerar Modelo de Chamado

- **Objetivo:** Produzir um registro estruturado de passagem ou atendimento para copiar.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/modelo-chamados`. 2. Na aba Modelo, preencher os campos obrigatórios. 3. Copiar o texto gerado. 4. Na aba Chamado, preencher o registro, opcionalmente incluir a conversa e limpar os campos.
- **Resultado esperado:** campos obrigatórios vazios impedem a cópia e são destacados; o texto gerado contém os blocos preenchidos; a conversa de WhatsApp só entra quando o checkbox está marcado; a limpeza remove somente os campos da aba de atendimento.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; o gerador é local e não envia a conversa para um serviço externo.
- **Rotas/endpoints/tabelas:** `/modelo-chamados`; nenhum endpoint/tabela; todo o processamento é local.
- **Status/automação:** **cenário documentado**; não há teste automatizado do gerador.
### CF-09 — Consultar Stack e Políticas

- **Objetivo:** Oferecer referências institucionais e técnicas em páginas somente leitura.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/base-conhecimento?aba=stack` (ou `/stack`, que redireciona) e expandir/recolher cada aplicação. 2. Abrir `/politica` e navegar pelas seções.
- **Resultado esperado:** Stack apresenta as três seções do ecossistema; Política apresenta as regras internas. Alterações de conteúdo não são enviadas ao backend.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; as páginas são somente leitura e usam conteúdo estático.
- **Rotas/endpoints/tabelas:** `/base-conhecimento` (aba Stack), `/stack` (redirect) e `/politica`; nenhum endpoint/tabela; `stack.component.ts`, `politica.component.ts` e `politica.data.ts`.
- **Status/automação:** **cenário documentado**; não há testes dessas páginas.
### CF-10 — Consultar Procedimentos e Utilidades da Visão ADM

- **Objetivo:** Reunir, em uma tela, a busca de procedimentos administrativos e a central de utilidades da JCA.
- **Pré-condições:** operador autenticado; dados estáticos carregados.
- **Passos:** 1. Abrir `/visao-adm`. 2. Pesquisar procedimento e filtrar por setor. 3. Abrir um procedimento. 4. Na área de utilidades, pesquisar, filtrar categoria, ordenar, alternar grade/lista, favoritar e acessar um sistema.
- **Resultado esperado:** procedimentos respeitam busca/setor e abrem detalhe; favoritos, acessos e últimos utilizados são mantidos localmente. A ordenação e os contadores não chamam o backend.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; os dados de favoritos/últimos utilizados dependem do `localStorage` do navegador.
- **Rotas/endpoints/tabelas:** `/visao-adm`; nenhum endpoint/tabela; `procedimentos.data.ts`, `utilidades.data.ts` e `localStorage` (`cc.adm.utilidades.favoritos`/`cc.adm.utilidades.runtime`).
- **Status/automação:** **cenário documentado**; não há teste automatizado da Visão ADM.
### CF-11 — Abrir e imprimir um Procedimento

- **Objetivo:** Ler um procedimento administrativo completo e imprimir a página.
- **Pré-condições:** operador autenticado; ID presente em `procedimentos.data.ts`.
- **Passos:** 1. Abrir `/visao-adm/detalhe/:id`. 2. Expandir Objetivo, Pré-requisitos, Passos, Erros, FAQ, Vídeo, Anexos e Histórico. 3. Clicar em `Imprimir`.
- **Resultado esperado:** o procedimento selecionado é renderizado, as seções alternam e `window.print()` é acionado; ID inválido mostra “Procedimento não encontrado” com retorno à Visão ADM.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a impressão depende de o navegador permitir a chamada a `window.print()`.
- **Rotas/endpoints/tabelas:** `/visao-adm/detalhe/:id`; nenhum endpoint/tabela; `ProcedimentosService` local.
- **Status/automação:** **cenário documentado**; não há teste automatizado do detalhe.
### CF-13 — Consultar o FAQ Sistema (catálogo de telas)

- **Objetivo:** Responder, em cards por tela do sistema, o que cada tela faz (objetivo), o que ela usa (endpoints/tabelas), a história passo a passo e o estado atual — com badge de status.
- **Pré-condições:** operador autenticado (tela herda `authGuard` do shell); nenhuma outra — o item de menu não tem `roles` e é aberto a todo operador autenticado.
- **Passos:** 1. Abrir `/faq-sistema` pelo menu **Conhecimento → Guia do Sistema**, ou digitar a rota. 2. Usar a busca (tela/título/rota/grupo/objetivo/endpoints) e os chips de filtro por grupo e por status. 3. Expandir/recolher um card individual, ou todos de uma vez. 4. Usar o **mini sistema** (`abrirNoGrupo`) para ir direto ao card de um módulo: filtra por módulo, abre/rola o card. 5. Conferir o badge de versão (SemVer `v{{ versao }}`, commit e data local) exibido abaixo do `PageHeader`.
- **Resultado esperado:** os cards mostram apenas as telas do filtro vigente com a contagem atualizada; expandir um card revela objetivo, pré-condições, passos, resultado, estado atual e badge de status colorido (ok/parcial/pendente). O badge de versão exibe o SemVer, o commit e a data de deploy; o mini sistema navega por módulo sem limpar o restante dos filtros.
- **Estado atual/bloqueio:** sem limitação da tela — a **fonte do dado é v1** (estática): `historiasTelas` de `historias-telas.data.ts`, gerado por `extract-historias-telas.ts` com fonte repontada para **este arquivo** (fase 5 do plano docs-api); o catálogo exibe as histórias do `data.ts` regenerado, incluindo a própria CF-13. A evolução v2 prevista é trocar a fonte estática por `GET /api/v1/docs` lendo de `tbdoc_doc`, no service (não no template). Quando `GET /version.json` falha, o fallback do `BuildInfoService` é `APP_VERSION` (`1.0.0`).
- **Rotas/endpoints/tabelas:** `/faq-sistema`; v1 nenhum endpoint/tabela (dado local); v2 prevista `GET /api/v1/docs` sobre `tbdoc_doc`.
- **Status/automação:** **cenário documentado**; existe `faq-sistema.component.spec.ts` com provider mock de `BuildInfoService` e testes para o carregamento de versão/commit/data e para `abrirNoGrupo`; verificado por `ng build`, sem cobertura de comportamento de render.
