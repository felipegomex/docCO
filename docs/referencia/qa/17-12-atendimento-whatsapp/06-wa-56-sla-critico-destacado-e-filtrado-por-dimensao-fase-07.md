---
id: 17-12-atendimento-whatsapp.06
titulo: "WA-56 — SLA crítico destacado e filtrado por dimensão (Fase 07) (+6)"
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
tokens_estimados: 3119
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-56 — SLA crítico destacado e filtrado por dimensão (Fase 07)

- **Objetivo:** achar rapidamente o que está perto de vencer, com primeira resposta e resolução avaliadas **separadamente**.
- **Pré-condições:** metas e horário útil configurados; percentual de aviso (padrão 80); cards em situações diferentes.
- **Passos:** 1. Abrir `/whatsapp`. 2. Escolher **1ª resposta: Crítico** e conferir os cards. 3. Trocar para **Resolução: Estourado**. 4. Passar o mouse no SLA de um card crítico.
- **Resultado esperado:** o card em `critico` aparece destacado (classe de SLA e tooltip). Os filtros usam **a dimensão escolhida** (`primeiraRespostaSlaSituacao` ou `resolucaoSlaSituacao`) e são server-side; a classificação acontece em memória **antes** de contar/paginar, com o mesmo `IWhatsAppSlaService` do painel. O `slaSituacao` do card continua sendo a **geral** (pior caso entre as duas). Sem meta/expediente a situação é `null` e o card mostra o tempo sem veredito.
- **Estado atual/bloqueio:** o SLA não é traduzível em SQL (depende de expediente/espera do cliente); por isso o caminho com filtro de SLA carrega os ids candidatos e aplica `AplicarIds` com lista literal.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos?primeiraRespostaSlaSituacao=&resolucaoSlaSituacao=`; `tbwhatsapp_sla_config`, `tbwhatsapp_sla_horario`, `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário documentado**; coberto por `Filtro_de_primeira_resposta_usa_a_dimensao_correta`, `Filtro_de_resolucao_usa_a_dimensao_correta` e `Card_traz_empresa_etiquetas_investigacao_e_as_duas_situacoes`.
### WA-57 — Ações do card por status e permissão (Fase 07)

- **Objetivo:** oferecer a ação certa para o estado do card e explicar por que as demais não se aplicam.
- **Pré-condições:** cards em “Não atendido”, “Em atendimento”, “Aguardando cliente”, “Resolvido” e “Finalizado”; um operador comum e um administrador.
- **Passos:** 1. Abrir o menu de ações de cada card. 2. Passar o mouse nas ações desabilitadas. 3. Como operador comum, tentar **Arquivar** um finalizado. 4. Como administrador, repetir.
- **Resultado esperado:** `acoesDoCard` (em `whatsapp.utils.ts`) libera **Assumir** só sem responsável; **Transferir** e **Devolver** exigem responsável; **Aguardar cliente** só em atendimento; **Retomar** só aguardando; **Marcar como visto** só em card aberto (0/1/2); **Resolver** fora de resolvido/finalizado/visto; **Reabrir** em resolvido **ou visto**; **Finalizar** só resolvido; **Arquivar** só finalizado **e** administrador. Em **Visto**, a única ação habilitada é **Reabrir**. Ação desabilitada mostra o motivo no `title`/toast. O backend revalida (409 de estado, 403 de permissão) e o quadro recarrega após a mutação.
- **Estado atual/bloqueio:** a regra da tela é **espelho** de `AtendimentoService`; se o backend mudar, o menu precisa mudar junto. Arquivar remove do quadro operacional, mas o atendimento continua pesquisável.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `POST /api/v1/whatsapp/atendimentos/{id}/{assumir|transferir|devolver|resolver|reabrir|aguardar-cliente|retomar|visto|finalizar|arquivar}`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; coberto por `whatsapp.utils.spec.ts` (matriz de ações) e pelo spec do card.
### WA-58 — Hub administrativo reúne indicadores e atalhos (Fase 08)

- **Objetivo:** o gestor do canal ter um ponto de entrada único para os indicadores e as telas administrativas, sem indicador calculado na tela.
- **Pré-condições:** usuário com perfil **Administrador**; módulo WhatsApp habilitado.
- **Passos:** 1. Entrar com administrador e abrir o submenu WhatsApp: o item **Admin** aparece. 2. Abrir `/whatsapp/admin`. 3. Conferir os quatro indicadores e os atalhos. 4. Repetir o login com operador comum. 5. Chamar `GET /api/v1/whatsapp/admin/contatos` diretamente sem perfil.
- **Resultado esperado:** o hub mostra **Atendimentos ativos** (`metricas.emAndamento`), **Fila sem responsável** (total de `GET /atendimentos?semResponsavel=true`), **Números conectados** (quantos estão `ready` em `GET /saude`) e **Erros na fila de webhook** (`GET /monitor`). Os atalhos levam a `/whatsapp/sla`, `/whatsapp/etiquetas` e `/whatsapp/admin/contatos`; "Configuração avançada" leva a números, respostas rápidas, retenção, auditoria e configuração IA. **Nenhum número é calculado no cliente**; a falha de um endpoint deixa só o card correspondente sem valor. O item **Admin** fica oculto para operador comum e todas as rotas `/admin` respondem **`403`** sem perfil administrativo — esconder o menu não é controle de acesso.
- **Estado atual/bloqueio:** o hub **não duplica** as rotas existentes; ele só aponta para elas. Os indicadores de endpoints administrativos podem vir vazios para quem não tem o perfil, e o hub trata isso como ausência de valor, não como erro fatal.
- **Rotas/endpoints/tabelas:** `/whatsapp/admin`; `GET /api/v1/whatsapp/metricas`, `GET /api/v1/whatsapp/atendimentos?semResponsavel=true`, `GET /api/v1/whatsapp/saude`, `GET /api/v1/whatsapp/monitor`, `GET /api/v1/whatsapp/admin/contatos`; `tbwhatsapp_atendimento`, `tbwhatsapp_numero`, `tbwhatsapp_webhook_fila`.
- **Status/automação:** **cenário documentado**; `whatsapp-admin.component.spec.ts` cobre os indicadores, a tolerância à falha de um endpoint e os atalhos.
### WA-59 — Filtros e paginação de contatos no servidor (Fase 08)

- **Objetivo:** recortar a base inteira de contatos, e não apenas a página carregada, com o total do recorte.
- **Pré-condições:** contatos identificados e pendentes, de mais de uma empresa, e ao menos um telefone com máscara; administrador autenticado.
- **Passos:** 1. Abrir `/whatsapp/admin/contatos`. 2. Buscar por nome, `chatId` e telefone. 3. Alternar **Todos/Identificados/Pendentes**. 4. Escolher uma empresa no typeahead (mínimo 2 caracteres). 5. Digitar o telefone com e sem máscara. 6. Combinar os filtros e trocar de página.
- **Resultado esperado:** os filtros vão na query de `GET /admin/contatos` e são aplicados **antes de contar e paginar**: `totalCount`/`totalPages` são do recorte. `identificados=true` traz só `empresaId != null`; `false`, só pendentes; `empresaId` restringe a empresa; `telefone` compara **apenas os dígitos** com `WAT_TelefoneNormalizado` (com ou sem máscara dá o mesmo resultado). A busca cobre nome, `chatId`, número e telefone normalizado. A tabela exibe total de atendimentos e último atendimento por contato, calculados por **subconsultas correlacionadas na mesma query paginada** (sem N+1).
- **Estado atual/bloqueio:** o `GET /whatsapp/contatos` do overlay **não** foi alterado; esta é uma rota nova. A ordenação é pelo último atendimento, com contatos sem atendimento no fim.
- **Rotas/endpoints/tabelas:** `/whatsapp/admin/contatos`; `GET /api/v1/whatsapp/admin/contatos` (`busca`, `identificados`, `empresaId`, `telefone`, `pagina`, `tamanho`); `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbcliente`.
- **Status/automação:** **cenário documentado**; `WhatsAppAdminContatoTests` cobre cada filtro isolado e combinado, paginação (conferindo a **segunda página**), agregação sem N+1, a **ordenação/paginação traduzida para SQL Server** (`ORDER BY`/`OFFSET`/`FETCH`, sem `OPENJSON`) e o spec da página cobre os filtros enviados ao serviço.
### WA-60 — Editar contato no painel e tratar conflito 409 (Fase 08)

- **Objetivo:** editar empresa, tipo e observação fora do atendimento, sem sobrescrever a alteração de outro operador.
- **Pré-condições:** contato listado; administrador autenticado; um segundo operador para provocar o conflito.
- **Passos:** 1. Em `/whatsapp/admin/contatos`, clicar em **Editar** na linha. 2. Conferir a ficha no painel lateral (mesmo formulário do atendimento). 3. Salvar sem fechar a página. 4. Com a ficha aberta em outra sessão, alterar o mesmo contato e tentar salvar a ficha antiga.
- **Resultado esperado:** o painel reutiliza `VincularEmpresaModalComponent` no modo `painel`, com typeahead de empresas (`GET /contatos/empresas`), tipo de contato e observação; a escrita é o **`PATCH /contatos/{id}` da Fase 01**, com a `versao` da ficha (lock otimista) e auditoria. Ao salvar, a lista é recarregada e um aviso confirma a atualização. Em **`409`** `CONFLICT_CONTATO_VERSAO`, a ficha devolvida pelo backend **substitui** a que estava aberta e a tela avisa que outro operador atualizou; o operador revisa antes de salvar de novo.
- **Estado atual/bloqueio:** **não existe endpoint de escrita novo** nesta fase; vincular contato pendente continua permitido ao atendente no atendimento, e a troca/remoção de vínculo de identificado é administrativa (WA-39). Nenhuma empresa é criada pelo WhatsApp.
- **Rotas/endpoints/tabelas:** `/whatsapp/admin/contatos`; `GET /api/v1/whatsapp/contatos/{id}`, `GET /api/v1/whatsapp/contatos/empresas`, `PATCH /api/v1/whatsapp/contatos/{id}`; `tbwhatsapp_contato` (`WAT_EmpresaId`, `WAT_TipoContato`, `WAT_Observacao`, `WAT_Versao`), `tbcliente`.
- **Status/automação:** **cenário documentado**; o spec da página cobre abrir a ficha, salvar/recarregar e o `409` mantendo a ficha devolvida; `WhatsAppAdminContatoTests` cobre a listagem administrativa.
### WA-61 — Histórico do contato em painel com link para o atendimento (Fase 08)

- **Objetivo:** consultar todos os ciclos de atendimento de um contato a partir da gestão administrativa e abrir o atendimento correspondente.
- **Pré-condições:** contato com mais de um atendimento (inclusive reaberturas); administrador autenticado.
- **Passos:** 1. Em `/whatsapp/admin/contatos`, clicar em **Histórico**. 2. Conferir a gaveta lateral. 3. Clicar em um atendimento. 4. Abrir o histórico de um contato sem atendimentos.
- **Resultado esperado:** `GET /admin/contatos/{id}/historico` devolve os atendimentos do contato **mais recente primeiro**, com status, motivo, fila/número, responsável, abertura, resolução e reaberturas. Cada item leva a `/whatsapp/atendimento/{id}`. Sem atendimentos, a gaveta mostra estado vazio; falha na leitura mostra erro com nova tentativa. Contato inexistente → **`404`**.
- **Estado atual/bloqueio:** o histórico usa a mesma regra do `HistoricoCliente` (todas as conversas do contato) e é **somente leitura**; a edição abre no outro painel e fecha o histórico (e vice-versa).
- **Rotas/endpoints/tabelas:** `/whatsapp/admin/contatos`, `/whatsapp/atendimento/{id}`; `GET /api/v1/whatsapp/admin/contatos/{id}/historico`; `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_numero`, `tboperador`.
- **Status/automação:** **cenário documentado**; `WhatsAppAdminContatoTests` cobre o histórico completo, a não inclusão de atendimento de outro contato e o `404`; o spec da página cobre a gaveta e o erro com nova tentativa.
### WA-62 — Contato `@lid` marcado como identificador de privacidade (Fase 08)

- **Objetivo:** o gestor não confundir o identificador de privacidade do WhatsApp (`@lid`) com um telefone nem tentar tratá-lo como duplicidade.
- **Pré-condições:** contato cujo `chatId` termina em `@lid`; administrador autenticado.
- **Passos:** 1. Em `/whatsapp/admin/contatos`, localizar o contato `@lid`. 2. Conferir o badge e o telefone. 3. Abrir o histórico.
- **Resultado esperado:** a linha traz o badge **`@lid`** ("Identificador de privacidade do WhatsApp") e `ehLid=true`, com telefone nulo; a listagem e o histórico exibem o **aviso** de que o contato usa identificador de privacidade e **não é um telefone**. O `@lid` não é normalizado, não entra na chave de duplicidade e não aparece no histórico de outro telefone. O filtro de telefone não o encontra.
- **Estado atual/bloqueio:** a unificação manual continua sendo o caminho para associar um `@lid` a um contato real; não há conversão automática.
- **Rotas/endpoints/tabelas:** `/whatsapp/admin/contatos`; `GET /api/v1/whatsapp/admin/contatos`, `GET /api/v1/whatsapp/admin/contatos/{id}/historico`; `tbwhatsapp_contato.WAT_ChatId`/`WAT_TelefoneNormalizado`.
- **Status/automação:** **cenário documentado**; `WhatsAppAdminContatoTests` cobre o `@lid` marcado e fora da normalização; o spec da página cobre o badge `@lid`.
