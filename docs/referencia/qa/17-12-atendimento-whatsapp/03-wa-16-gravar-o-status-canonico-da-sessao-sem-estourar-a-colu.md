---
id: 17-12-atendimento-whatsapp.03
titulo: "WA-16 — Gravar o status canônico da sessão sem estourar a coluna (+8)"
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
tokens_estimados: 3409
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-16 — Gravar o status canônico da sessão sem estourar a coluna

- **Objetivo:** o evento de sessão atualizar o número **sem** erro de truncamento e **sem** deixar a linha presa na fila.
- **Pré-condições:** número cadastrado com sessão no gateway; base com `tbwhatsapp_numero.WAT_Status` em **`varchar(50)`** (aplicar `Migrations/Sql/WhatsApp_Idempotente.sql`, que carrega o bloco idempotente de ajuste de largura).
- **Passos:** 1. Parear a sessão e ler o QR. 2. Conferir `tbwhatsapp_numero` e `tbwhatsapp_webhook_fila`.
- **Resultado esperado:** `WAT_Status` = **`ready`** (status **canônico** — **nunca** o nome do evento `session.authenticated`), `WAT_Numero` preenchido com o telefone do pareamento e a linha da fila em `WAT_Estado = 2` (processado). **Nunca** erro **8152** ("dados seriam truncados") no log do SQL Server.
- **Estado atual/bloqueio:** o defeito original gravava o **nome do evento** (`session.authenticated`, 21 caracteres) em `varchar(20)`: o SQL Server recusava o INSERT/UPDATE com o **erro 8152**, a linha da fila ficava **presa em "processando"** e o **telefone do número nunca era gravado**. A causa **não** era a fila — era o nome do evento em coluna curta. Correção: `StatusDaSessao` grava sempre um status canônico, o modelo passou a `[MaxLength(50)]` e o script alarga a coluna com bloco idempotente. ⚠️ **O script precisa ser reaplicado**: código novo sem script novo mantém o `varchar(20)` e o erro volta. Diagnóstico em [`troubleshooting.md`](../../../operacao/troubleshooting.md) §19.10.1.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `GET /api/v1/whatsapp/numeros`, `GET /api/v1/whatsapp/saude`; `tbwhatsapp_numero` (`WAT_Status`, `WAT_Numero`), `tbwhatsapp_webhook_fila`.
- **Status/automação:** coberto por teste unitário — `WhatsAppStatusSessaoTests` (mapeamento evento → status, limite de 50 caracteres e gravação do telefone).
### WA-17 — Recuperar a linha da fila presa em "processando"

- **Objetivo:** um evento **não se perder** quando o backend morre no meio do processamento.
- **Pré-condições:** backend publicado; fila com pelo menos um evento aguardando processamento.
- **Passos:** 1. Forçar a queda no meio do processamento (redeploy ou recycle do pool do IIS com a fila ocupada). 2. Conferir a fila. 3. Conferir o resultado no Kanban.
- **Resultado esperado:** a linha que estava em `WAT_Estado = 1` **volta para `Pendente` sozinha** — **todas** na largada do worker e, a cada **30 s**, as que estão presas há mais de **2 min** — e o evento é processado (o atendimento nasce). Para forçar na mão: `UPDATE tbwhatsapp_webhook_fila SET WAT_Estado = 0, WAT_Tentativas = 0, WAT_ProximoProcessamentoEm = NULL, WAT_Erro = NULL WHERE WAT_Estado = 1;`.
- **Estado atual/bloqueio:** o **comentário do worker prometia** a volta automática e o **código não fazia**: o evento ficava perdido para sempre. Foi assim que o telefone do número ficou sem registro — o recycle do publish caiu em cima do evento `session.authenticated`. O reprocessamento é **idempotente** (índice único `(WAT_ChatId, WAT_ExternalId)` de `tbwhatsapp_mensagem`).
- **Rotas/endpoints/tabelas:** `tbwhatsapp_webhook_fila` (`WAT_Estado`, `WAT_Tentativas`, `WAT_ProximoProcessamentoEm`, `WAT_Erro`); procedimento em [`instalacao-openwa.md`](../../../operacao/instalacao-openwa/INDEX.md) §9.4.
- **Status/automação:** coberto por teste unitário — `WhatsAppFilaRecuperacaoTests` (linha presa volta para a fila; linha pendente não é tocada).
### WA-18 — Consultar a saúde do canal com a sessão ativa

- **Objetivo:** a tela de saúde do canal responder com o estado real da sessão, **depois** que ela fica ativa.
- **Pré-condições:** sessão do gateway em `ready` (pareada); `WhatsApp__BaseUrl` e `WhatsApp__ApiKey` válidas.
- **Passos:** 1. Abrir `/whatsapp/numeros`. 2. Conferir o bloco de saúde (gateway acessível e status do gateway).
- **Resultado esperado:** `GET /api/v1/whatsapp/saude` devolve `200` com `gatewayAcessivel: true` e `statusGateway: ready` — **nunca** "O gateway WhatsApp devolveu uma resposta inesperada".
- **Estado atual/bloqueio:** `OpenWaSessaoDto.LastActive` era `long?` e o gateway devolve **data ISO-8601** (`"2026-06-25T09:01:55.000Z"`): a desserialização estourava e a tela falhava **só depois** que a sessão ficava ativa (com a sessão `created`, tudo funcionava). O campo passou a `string?`. Diagnóstico em [`troubleshooting.md`](../../../operacao/troubleshooting.md) §19.10.3.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `GET /api/v1/whatsapp/saude`; no gateway `GET /api/sessions/{id}`.
- **Status/automação:** **cenário documentado** — não há teste automatizado cobrindo a desserialização do `lastActive`.
### WA-19 — Ver a tela de números refletir o status sozinha

- **Objetivo:** a tela de números mostrar o status atual da sessão **sem** recarregar a página.
- **Pré-condições:** perfil **Administrador**; sessão do gateway pareada; `/whatsapp/numeros` aberta.
- **Passos:** 1. Abrir `/whatsapp/numeros`. 2. Parear a sessão em outro dispositivo (ou pedir o QR). 3. **Não recarregar** a página. 4. Esperar cerca de 15 s.
- **Resultado esperado:** a listagem **se atualiza sozinha a cada 15 s** (a atualização **pausa** com a aba oculta) e o status passa a `ready` sem reload; o painel do **QR fecha** quando a sessão sai de `qr_ready`.
- **Estado atual/bloqueio:** o estado da sessão muda **fora da Central** — o QR é lido no celular e o gateway só avisa por webhook. Antes, a tela carregava uma vez e ficava mostrando "aguardando QR" mesmo com o número já conectado (era preciso recarregar na mão). O polling do **Kanban** é de 30 s (`INTERVALO_ATUALIZACAO_MS`); o da **tela de números** é de **15 s**.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `GET /api/v1/whatsapp/numeros`, `GET /api/v1/whatsapp/saude`; `tbwhatsapp_numero`.
- **Status/automação:** **cenário documentado** — não há teste de frontend para o componente.
### WA-20 — Filtrar a fila por "sem responsável" (fila)

- **Objetivo:** o operador visualizar apenas os atendimentos que **não têm responsável** (a fila pura), sem os seus próprios nem os de outros.
- **Pré-condições:** operador autenticado; módulo habilitado; pelo menos um atendimento sem responsável na base.
- **Passos:** 1. Abrir `/whatsapp`. 2. Ativar o filtro "Sem responsável" (toggle da tela). 3. Conferir que só aparecem cards com `responsavelId` nulo.
- **Resultado esperado:** a query envia `semResponsavel=true` em `GET /api/v1/whatsapp/atendimentos`. O backend aplica o filtro mutuamente exclusivo com `responsavelId` — o card sem responsável aparece na coluna **Não atendido**, e a contagem da coluna reflete o filtro. O toggle **não** combina com "Só meus" (quando um está ligado, o outro desliga).
- **Estado atual/bloqueio:** sem limitação específica; o filtro é aplicado no backend (o frontend só envia o parâmetro).
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos?semResponsavel=true`; `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-21 — Acessar "Minhas Conversas" e abrir uma conversa

- **Objetivo:** o operador ver **apenas** os atendimentos sob sua responsabilidade e abrir a conversa ao lado.
- **Pré-condições:** operador autenticado; pelo menos um atendimento com `responsavelId` igual ao operador logado.
- **Passos:** 1. Abrir `/whatsapp/minhas-conversas` (item **Minhas Conversas** na seção *SUPORTE*). 2. Digitar termo na busca local (nome, telefone, assunto, fila, última mensagem). 3. Usar `Ctrl+K` para focar a busca. 4. Usar `Alt+1` a `Alt+9` para abrir a N-ésima conversa da lista **já filtrada e ordenada**. 5. Clicar em um item da lista.
- **Resultado esperado:** a lista carrega com `somenteMeus=true`, `tamanho=100` e o recorte padrão **Abertas** (`somenteAbertos=true` → status 0, 1 e 2). Ordenação client-side: **não lidas primeiro** → **SLA estourado/atrasado/no_prazo** → **última mensagem mais recente** → **prioridade alta**. O outlet `:id` carrega `AtendimentoWhatsappComponent` em modo **embutido** (`data: { embutido: true }`), sem cabeçalho próprio. `Ctrl+K` foca e seleciona o campo de busca; `Alt+digito` navega para `/whatsapp/minhas-conversas/:id`. Atualização a cada 30 s, pausa com aba oculta.
- **Estado atual/bloqueio:** sem limitação específica; a ordenação é client-side (sobre até 100 itens) — o backend não conhece a regra de "minhas conversas" ordenada.
- **Rotas/endpoints/tabelas:** `/whatsapp/minhas-conversas`, `/whatsapp/minhas-conversas/:id`; `GET /api/v1/whatsapp/atendimentos?somenteMeus=true&tamanho=100&somenteAbertos=true`; `tbwhatsapp_atendimento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-22 — Consultar o histórico do cliente (aba lateral "Cliente")

- **Objetivo:** ver **todos os ciclos** do cliente (métricas, lista de atendimentos e as mensagens da conversa) a partir do painel do atendimento.
- **Pré-condições:** atendimento aberto em `/whatsapp/atendimento/:id` ou `/whatsapp/minhas-conversas/:id`; operador autenticado.
- **Passos:** 1. No painel lateral do atendimento, clicar na aba **Cliente**. 2. Aguardar o carregamento. 3. Rolar a lista de atendimentos do cliente. 4. Ativar o toggle **Ver histórico completo** na conversa.
- **Resultado esperado:** ao abrir a aba, o frontend chama `GET /api/v1/whatsapp/atendimentos/{id}/historico-cliente`. A resposta traz: **contato** (nome, chatId, primeiro/último contato), **métricas** (total, resolvidos, reabertos, tempo médio de resolução, último operador), **atendimentos** (cards com status, motivo, fila, número, responsável, abertura, resolução, reaberturas) e **mensagens** (cronológicas, últimas 300 da conversa, com `atendimentoId` do ciclo). O toggle "Ver histórico completo" usa **a mesma resposta** — não faz nova chamada. A aba lateral e o toggle compartilham o signal `historico`.
- **Estado atual/bloqueio:** sem limitação específica; o histórico é **somente leitura** — não há ação de edição no painel do cliente. O backend devolve `404` tanto para "não existe" quanto para "sem permissão".
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; `GET /api/v1/whatsapp/atendimentos/{id}/historico-cliente`; `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-23 — Visualizar mídia da mensagem (imagem, áudio, vídeo, documento)

- **Objetivo:** abrir a mídia de uma mensagem do WhatsApp (foto, áudio, vídeo, PDF, etc.) sob demanda.
- **Pré-condições:** atendimento aberto com mensagem que tem `temMidia = true`; mídia não removida pela retenção.
- **Passos:** 1. Na conversa, localizar uma mensagem com ícone de clipe (imagem/áudio/vídeo/documento). 2. Clicar no botão de carregar (spinner enquanto busca). 3. Para imagem/vídeo: clicar em **Abrir em nova aba**. 4. Para documento: clicar em **Baixar**.
- **Resultado esperado:** o componente `MidiaMensagemComponent` chama `GET /api/v1/whatsapp/mensagens/{id}/midia` **apenas ao clicar** (nunca em massa). Retorna `Blob` com `Content-Type` validado pela whitelist do backend (`WhatsAppMidiaSeguranca`). Imagem/vídeo abre em object URL; áudio tem player nativo; documento tem botão download. Erro **404** exibe "Arquivo indisponível: pode ter sido removido pela política de retenção". Erro genérico exibe "Não foi possível carregar a mídia" com botão "Tentar de novo". A object URL é revogada no destroy do componente.
- **Estado atual/bloqueio:** sem limitação específica; a mídia é buscada **sob demanda** (não carrega automaticamente ao abrir a conversa). O backend aplica whitelist de MIME e header `X-Content-Type-Options: nosniff`.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; `GET /api/v1/whatsapp/mensagens/{id}/midia`; `tbwhatsapp_mensagem` (`WAT_MidiaBase64`, `WAT_MidiaMime`, `WAT_MidiaNome`).
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-24 — Visualizar o layout do Assistente IA (aba lateral "Assistente IA")

- **Objetivo:** confirmar a estrutura da aba **Assistente IA** (Fase 4 — layout "em breve") no painel lateral do atendimento.
- **Pré-condições:** atendimento aberto; operador autenticado.
- **Passos:** 1. No painel lateral, clicar na aba **Assistente IA**. 2. Observar as seções.
- **Resultado esperado:** a aba mostra: badge **Em breve** (roxo) + texto explicativo; **Resumo automático** (placeholder esmaecido); **Classificação** com 4 campos (Categoria, Prioridade, Sentimento, Intenção) — todos com valor "—"; **Sugestões** (Respostas, Artigos, Procedimentos, Próximos passos) — cada item com rótulo "Não disponível ainda". Todos os botões ficam **desabilitados** com `title="Em breve"`. **Nenhuma chamada de API** é feita. O componente recebe `atendimentoId` por `@Input` para uso futuro.
- **Estado atual/bloqueio:** layout intencionalmente vazio — o recurso está planejado para fase posterior. Não há endpoint nem serviço de IA no módulo WhatsApp atual.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; nenhum endpoint HTTP; componente `AssistenteIaPainelComponent`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
