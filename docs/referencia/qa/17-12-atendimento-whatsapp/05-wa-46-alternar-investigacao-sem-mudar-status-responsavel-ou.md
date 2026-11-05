---
id: 17-12-atendimento-whatsapp.05
titulo: "WA-46 — Alternar investigação sem mudar status, responsável ou SLA (Fase 06) (+9)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 3375
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-46 — Alternar investigação sem mudar status, responsável ou SLA (Fase 06)

- **Objetivo:** marcar o caso como "em investigação" sem confundir com o status do Kanban.
- **Pré-condições:** atendimento em atendimento, com responsável e SLA calculado.
- **Passos:** 1. Anotar status, responsável, SLA e datas do ciclo. 2. Na aba **Resumo**, acionar o toggle **Em investigação**. 3. Reabrir/atualizar e conferir os mesmos campos. 4. Desligar o toggle.
- **Resultado esperado:** `PATCH /atendimentos/{id}/investigacao` grava `emInvestigacao: true|false` e emite `investigacao_iniciada`/`investigacao_finalizada` **só quando o valor muda**. Status, responsável, SLA e datas de ciclo ficam **intactos** — não há quinta coluna no Kanban. O botão reflete `aria-pressed`/rótulo Ativado/Desativado.
- **Estado atual/bloqueio:** `InvestigacaoAtendimentoRequest` não aceita autor; o operador vem do token.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `PATCH /api/v1/whatsapp/atendimentos/{id}/investigacao`; `tbwhatsapp_atendimento.WAT_EmInvestigacao`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; coberto por `WhatsAppEtiquetaTests` (toggle preserva status/responsável/SLA).
### WA-47 — Observação restrita ao atendimento atual (Fase 06)

- **Objetivo:** registrar uma condição pontual que **não** aparece em outro atendimento do mesmo contato.
- **Pré-condições:** contato com pelo menos dois atendimentos; operador atendente.
- **Passos:** 1. No atendimento A, aba **Resumo**, salvar uma observação. 2. Abrir o atendimento B do mesmo contato. 3. Voltar ao A e limpar a observação (texto vazio). 4. Tentar salvar acima de 500 caracteres.
- **Resultado esperado:** `PATCH /atendimentos/{id}/observacao` grava `observacaoInterna` **somente no atendimento A**; B permanece `null`. Vazio/em branco limpa o campo; acima de 500 → `400`. O componente é distinto da **nota interna** (mensagem de direção `2`) e da **observação do cliente** (`tbwhatsapp_contato.WAT_Observacao`, Fase 01).
- **Estado atual/bloqueio:** o texto é do atendimento, não do contato; não há propagação entre ciclos do mesmo contato.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `PATCH /api/v1/whatsapp/atendimentos/{id}/observacao`; `tbwhatsapp_atendimento.WAT_ObservacaoInterna`.
- **Status/automação:** **cenário documentado**; coberto por `WhatsAppEtiquetaTests` (observação não vaza para outro atendimento).
### WA-48 — Auditoria da observação sem o texto (Fase 06)

- **Objetivo:** registrar que a observação mudou **sem** expor o conteúdo interno.
- **Pré-condições:** atendimento aberto; operador com acesso à auditoria (administrador) para conferir.
- **Passos:** 1. Salvar uma observação com texto identificável. 2. Conferir o evento no histórico do atendimento. 3. Abrir `/whatsapp/auditoria` e localizar a ação `atualizar_observacao`.
- **Resultado esperado:** o evento é `observacao_atualizada` com descrição genérica ("Observação interna atualizada.") e a auditoria registra a **ação** (`atualizar_observacao`) sem o texto no campo `alteracao`; o conteúdo não aparece em evento, auditoria ou log.
- **Estado atual/bloqueio:** a decisão D05 é aplicada no `AtendimentoService` (descrição fixa) e no controller (auditoria sem `alteracao`).
- **Rotas/endpoints/tabelas:** `PATCH /api/v1/whatsapp/atendimentos/{id}/observacao`, `GET /api/v1/whatsapp/auditoria` (admin); `tbwhatsapp_evento`, `tbwhatsapp_auditoria`.
- **Status/automação:** **cenário documentado**; coberto por `WhatsAppEtiquetaTests` (auditoria de observação sem o texto).
### WA-49 — Etiqueta inativa continua visível no atendimento (Fase 06)

- **Objetivo:** inativar uma etiqueta sem apagar o vínculo histórico.
- **Pré-condições:** atendimento com a etiqueta A aplicada; administrador para inativar o catálogo.
- **Passos:** 1. Como administrador, aplicar `PATCH /etiquetas/{id}` com `ativo: false`. 2. Reabrir o atendimento que já tinha A. 3. Abrir o seletor e tentar adicionar A a outro atendimento.
- **Resultado esperado:** A continua aparecendo como **chip** no atendimento que já a tinha (o detalhe devolve o vínculo histórico mesmo inativo), mas **não** entra em `disponiveis` e não pode ser adicionada a outro atendimento; removê-la do atendimento é permitido. A definição **não** é apagada e o nome fica livre para uma nova etiqueta ativa.
- **Estado atual/bloqueio:** `GET /etiquetas?somenteAtivas=true` (usado pelo seletor) não lista a inativa; o detalhe do atendimento inclui o vínculo mesmo assim. IDs inativos enviados para aplicação → `400`.
- **Rotas/endpoints/tabelas:** `PATCH /api/v1/whatsapp/etiquetas/{id}`, `GET /api/v1/whatsapp/etiquetas`, `PUT /api/v1/whatsapp/atendimentos/{id}/etiquetas`; `tbwhatsapp_etiqueta`, `tbwhatsapp_atendimento_etiqueta`.
- **Status/automação:** **cenário documentado**; coberto por `WhatsAppEtiquetaTests` e pelo spec de `EtiquetaSelectorComponent`.
### WA-50 — Operador recebe 403 ao gerenciar o catálogo (Fase 06)

- **Objetivo:** garantir que esconder o botão na tela não é controle de acesso.
- **Pré-condições:** operador autenticado **sem** papel/claim de administrador.
- **Passos:** 1. Chamar `GET /api/v1/whatsapp/etiquetas`. 2. Chamar `POST /api/v1/whatsapp/etiquetas` e `PATCH /api/v1/whatsapp/etiquetas/{id}` diretamente. 3. Conferir a tela.
- **Resultado esperado:** `GET` responde `200` (qualquer atendente lê o catálogo); `POST` e `PATCH` respondem **`403`** (`EhAdministrador` no controller — `IsInRole("Administrador")` ou claim `eh_admin == "true"` ou `perfil_id == "A"`). A tela não oferece a ação administrativa; um administrador completa as mesmas chamadas.
- **Estado atual/bloqueio:** aplicar etiqueta ao atendimento continua permitido ao atendente (`PUT /atendimentos/{id}/etiquetas`); o que é exclusivo do admin é a **definição** do catálogo (D06).
- **Rotas/endpoints/tabelas:** `GET/POST /api/v1/whatsapp/etiquetas`, `PATCH /api/v1/whatsapp/etiquetas/{id}`; `tbwhatsapp_etiqueta`.
- **Status/automação:** **cenário documentado**; coberto por `WhatsAppEtiquetaTests` (403 do operador, 201/200 do admin).
### WA-51 — Card do Kanban Service Desk com contexto completo (Fase 07)

- **Objetivo:** enxergar SLA, cliente, assunto, etiquetas, investigação, responsável e tempo **sem abrir** o atendimento.
- **Pré-condições:** atendimento com empresa vinculada, etiquetas aplicadas, investigação marcada, responsável e SLA calculado; operador com acesso ao canal.
- **Passos:** 1. Abrir `/whatsapp`. 2. Localizar o card na coluna do status. 3. Passar o mouse sobre o badge de SLA. 4. Abrir o menu de ações.
- **Resultado esperado:** o card (`KanbanServiceDeskCardComponent`) mostra o **SLA destacado** (rótulo/classe/ícone; `critico`/`atrasado`/`estourado` realçados), empresa (ou “Cliente pendente”), contato/grupo/solicitante, **assunto** (`motivo`; vazio cai para a última mensagem; depois “Sem assunto”), até **4 etiquetas + contador**, badge **Em investigação**, responsável e tempo. O tooltip do SLA traz as duas dimensões independentes e os minutos úteis. O card vem do `AtendimentoResumo` enriquecido (`empresaId`/`empresaNome`, `etiquetas`, `emInvestigacao`, `primeiraRespostaSlaSituacao`, `resolucaoSlaSituacao`) e as etiquetas da página são carregadas em **uma consulta agrupada** (sem N+1).
- **Estado atual/bloqueio:** o card é **apresentação + emissores**; a regra de ação vive em `acoesDoCard` e o backend revalida. O card **não** tem evento de arrasto.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos`; `tbwhatsapp_atendimento`, `tbwhatsapp_atendimento_etiqueta`, `tbwhatsapp_etiqueta`, `tbcliente`.
- **Status/automação:** **cenário documentado**; `WhatsAppKanbanServiceDeskTests` cobre o card completo e há spec de `KanbanServiceDeskCardComponent`.
### WA-52 — Cliente pendente no card e no filtro de empresa (Fase 07)

- **Objetivo:** distinguir, no próprio card, o contato **sem** empresa vinculada e não confundi-lo com cliente identificado.
- **Pré-condições:** atendimento de contato **sem** `WAT_EmpresaId`; um segundo atendimento com empresa.
- **Passos:** 1. Abrir `/whatsapp`. 2. Conferir o card do contato sem vínculo. 3. Filtrar por empresa (typeahead, mínimo 2 letras) escolhendo a empresa do outro card. 4. Remover o chip do filtro.
- **Resultado esperado:** o card pendente mostra **“Cliente pendente”** e `empresaId`/`empresaNome` nulos (`contatoIdentificado=false`). O filtro `empresaId` é **server-side** e devolve só os cards daquela empresa — o pendente fica de fora. O chip mostra o nome e o “x” limpa o recorte. A empresa continua vindo **do ERP em tempo real**; o WhatsApp não cadastra identidade.
- **Estado atual/bloqueio:** o typeahead só dispara a partir de 2 caracteres; empresa inativa não aparece.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos?empresaId=`, `GET /api/v1/whatsapp/contatos/empresas`; `tbwhatsapp_contato.WAT_EmpresaId`, `tbcliente`.
- **Status/automação:** **cenário documentado**; coberto por `Contato_sem_empresa_aparece_como_pendente` e `Filtro_de_empresa_limita_cards_e_total` (backend).
### WA-53 — Etiquetas no card: até 4 + contador (Fase 07)

- **Objetivo:** ver a classificação operacional no card sem que muitas etiquetas estourem o layout.
- **Pré-condições:** atendimento com **mais de 4** etiquetas aplicadas; uma delas pode estar inativa.
- **Passos:** 1. Abrir `/whatsapp`. 2. Conferir o bloco de etiquetas do card. 3. Passar o mouse no contador e na etiqueta inativa.
- **Resultado esperado:** aparecem no máximo **4** etiquetas, na ordem de nome devolvida pelo backend, e o excedente vira **“+N”** com tooltip. Etiqueta inativa continua visível como chip com aviso de “vínculo histórico”. O contador não abre modal nem navega — é indicador.
- **Estado atual/bloqueio:** o limite é `LIMITE_ETIQUETAS_CARD`/`resumirEtiquetas` em `whatsapp.utils.ts`; a consulta agrupada **não** filtra `Ativo`, para preservar o histórico. O filtro por etiqueta usa `EXISTS` no servidor.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos`; `tbwhatsapp_atendimento_etiqueta`, `tbwhatsapp_etiqueta`.
- **Status/automação:** **cenário documentado**; `whatsapp.utils.spec.ts` cobre `resumirEtiquetas` e `WhatsAppKanbanServiceDeskTests` cobre a ordem/consulta agrupada.
### WA-54 — Badge de investigação não muda a coluna (Fase 07)

- **Objetivo:** marcar o caso como investigação **sem** criar uma quinta coluna nem mexer no status.
- **Pré-condições:** atendimento em “Não atendido” ou “Em atendimento” com `emInvestigacao=true`.
- **Passos:** 1. Abrir `/whatsapp`. 2. Conferir que o card continua na coluna do **status**. 3. Ligar o filtro **Em investigação**. 4. Desligar e conferir as colunas.
- **Resultado esperado:** o badge **Em investigação** aparece no card, mas o `status` e a coluna permanecem os mesmos. O filtro `emInvestigacao=true` é server-side e devolve só os cards marcados; o quadro continua com **4 colunas operacionais** (Finalizado/Arquivado só com “incluir arquivados”). Não existe coluna “Investigação”.
- **Estado atual/bloqueio:** a investigação é atributo booleano do atendimento (Fase 06); ligar/desligar segue a regra de WA-46 (evento só quando muda).
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos?emInvestigacao=true`; `tbwhatsapp_atendimento.WAT_EmInvestigacao`.
- **Status/automação:** **cenário documentado**; coberto por `Filtro_de_investigacao_nao_muda_a_coluna_do_card` (backend).
### WA-55 — Filtro server-side atualiza total e paginação (Fase 07)

- **Objetivo:** o recorte escolhido valer para a **fila inteira**, e não só para os cards já carregados.
- **Pré-condições:** fila com mais cards do que o tamanho da página (`TAMANHO_PAGINA = 30`); pelo menos duas empresas/etiquetas/prioridades.
- **Passos:** 1. Abrir `/whatsapp` e anotar contagem e paginação de uma coluna. 2. Escolher **prioridade**, **empresa**, **etiqueta**, **investigação** ou uma situação de **SLA**. 3. Conferir o contador da coluna e as páginas. 4. Trocar a **ordem** no select e observar o total.
- **Resultado esperado:** cada coluna refaz `GET /atendimentos` com os parâmetros e o backend aplica o recorte **antes** de `CountAsync`/`Skip`/`Take`: `totalCount` e `totalPages` são do recorte. A **ordem** (`recentes`/`prioridade`/`tempo`) é local e **não** muda contagem — o select avisa “ordem desta página”. Situação de SLA fora de `no_prazo`/`critico`/`atrasado`/`estourado` devolve **400**.
- **Estado atual/bloqueio:** “Limpar” zera todos os filtros e volta à página 1. O chip de empresa é o único com nome legível; os demais são selects. O SLA precisa de expediente/meta para classificar.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos` (`prioridade`, `empresaId`, `etiquetaId`, `emInvestigacao`, `primeiraRespostaSlaSituacao`, `resolucaoSlaSituacao`); `tbwhatsapp_atendimento`, `tbwhatsapp_contato`, `tbwhatsapp_atendimento_etiqueta`.
- **Status/automação:** **cenário documentado**; coberto por `Filtro_de_prioridade_e_server_side`, `Total_e_paginacao_refletem_o_recorte_filtrado`, `Combinacao_de_filtros_nao_duplica_nem_perde_card` e `Situacao_de_sla_desconhecida_e_recusada`.
