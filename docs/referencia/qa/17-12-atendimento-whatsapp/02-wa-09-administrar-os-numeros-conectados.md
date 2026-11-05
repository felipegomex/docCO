---
id: 17-12-atendimento-whatsapp.02
titulo: "WA-09 — Administrar os números conectados (+6)"
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
tokens_estimados: 3366
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-09 — Administrar os números conectados

- **Objetivo:** cadastrar, conectar, acompanhar e desativar os números do canal.
- **Pré-condições:** perfil **Administrador** (o item de menu tem `roles: ['Administrador']`; o backend responde `403` para os demais, e a tela mostra mensagem própria em vez de erro genérico). A listagem e a saúde são de qualquer atendente.
- **Passos:** 1. Abrir `/whatsapp/numeros`. 2. Cadastrar um número (descrição, **setor ativo da `tbfuncao`**, mensagem de encerramento). 3. Ler o QR e conectar o celular. 4. Iniciar, parar e reconectar a sessão. 5. Conferir a saúde e a coluna **Setor**. 6. Remover o número.
- **Resultado esperado:** o `POST /numeros` **cria o cadastro e a sessão no gateway** (o servidor monta o nome `central-wa-<id>` — o cliente **não** manda id de sessão), **inicia a sessão** e **registra o webhook** com o segredo e os 9 eventos consumidos. O setor (`funcaoId`) é **obrigatório** e a lista do formulário traz **somente setores ativos**; a resposta devolve `funcaoId` e `fila` = `tbfuncao.DESCRICAO` (derivada, não editável). O QR vem como data URL e **não vai para log**. A tela mostra o espelho da Central (status e última mensagem) **e** o que o gateway responde agora (`statusGateway`, `ultimoErro`, `gatewayAcessivel`). Remover segue a regra de histórico do WA-14: sem histórico apaga o número fisicamente; com histórico, `409 NUMERO_COM_HISTORICO` sem `forcar` e cascata com `?forcar=true`.
- **Estado atual/bloqueio:** o gateway OpenWA v0.25.0 **expõe** a rota de exclusão de sessão (`DELETE /api/sessions/{id}`, verificado no gateway da VM) — um comentário antigo no código dizia o contrário e foi corrigido. Se o gateway **não** tiver a rota (ou estiver fora do ar), a Central **cai para o `stop`** e a desativação do cadastro **continua** — a falha do gateway não impede a desativação administrativa. A criação que falha no gateway **deixa o número cadastrado** com status `failed`, para o administrador reconectar depois.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `GET/POST /api/v1/whatsapp/numeros`, `PATCH/DELETE /api/v1/whatsapp/numeros/{id}`, `POST /api/v1/whatsapp/numeros/{id}/{iniciar|parar|reconectar}`, `GET /api/v1/whatsapp/numeros/{id}/qr`, `GET /api/v1/whatsapp/saude`; `tbwhatsapp_numero`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-10 — Consultar os indicadores do atendimento

- **Objetivo:** ler volume, tempos, fila e falhas do período escolhido.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/whatsapp/metricas` (ou o item **Indicadores do WhatsApp**). 2. Definir o período e, se quiser, o número, o responsável ou a **etiqueta** (inclusive inativa). 3. Conferir os cartões, as listas e o bloco **“Atendimentos por etiqueta”**.
- **Resultado esperado:** tudo vem **pronto do servidor** (`GET /api/v1/whatsapp/metricas`): total recebido, em andamento, resolvidos, reaberturas, transferências, falhas de envio, **média e mediana** do tempo até a 1ª resposta e do tempo de resolução (em minutos), fila de não atendidos e idade do mais antigo, além das contagens por status, por operador, por número, por dia e **por etiqueta** (`quantidade`, `tempoMedioResolucaoMinutos` e `taxaReabertura`). O período padrão é de **30 dias**.
- **Estado atual/bloqueio:** o tempo em **Aguardando cliente** é **descontado** das duas durações (a espera é do cliente). "Recebido" e "resolvido" olham **janelas diferentes**: um atendimento aberto antes do período e resolvido dentro dele conta como resolvido, não como recebido. O frontend **não recalcula** nenhuma média, mediana ou contagem — e as **metas** de SLA vêm do **backend/banco** (Fase 3): a tela mostra o tempo e a situação devolvida pelo servidor. O bloco por etiqueta sofre a mesma regra de não somabilidade descrita em WA-88.
- **Rotas/endpoints/tabelas:** `/whatsapp/metricas`; `GET /api/v1/whatsapp/metricas`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento`, `tbwhatsapp_mensagem`.
- **Status/automação:** **cenário documentado**; não há teste automatizado.
### WA-11 — Cadastrar o número e obter o QR sem erro

- **Objetivo:** o cadastro do número concluir **sem `404` nem `400` do gateway**, e o QR ficar disponível em seguida.
- **Pré-condições:** perfil **Administrador**; módulo habilitado no backend; `WhatsApp__BaseUrl` apontando para o gateway (`http://192.168.2.253:2785/api`) e `WhatsApp__ApiKey` válida.
- **Passos:** 1. Abrir `/whatsapp/numeros`. 2. Cadastrar um número (descrição, **setor ativo**, mensagem de encerramento). 3. Pedir o QR do número recém-cadastrado.
- **Resultado esperado:** o `POST /numeros` devolve `201` com o número **já com a sessão iniciada** (o status do gateway **não** é `created`: o `CriarAsync` cria **e inicia** a sessão). O `GET /numeros/{id}/qr` devolve `200` com o QR em data URL — **nunca** "O gateway WhatsApp respondeu 400". No gateway, `GET /api/sessions/{id}` mostra a sessão iniciada.
- **Estado atual/bloqueio:** dois defeitos do piloto moravam neste passo: o `BaseUrl` **sem barra final** mandava o pedido para `/sessions` e o gateway respondia `404` numa rota que existe (§19.9.1); e a sessão **não era iniciada**, e o QR do OpenWA só existe **depois** do `start`, então o gateway respondia `400` (§19.9.3). Se o `start` falhar, o **cadastro não é desfeito**: o número fica registrado e o administrador pode iniciar à mão.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `POST /api/v1/whatsapp/numeros`, `GET /api/v1/whatsapp/numeros/{id}/qr`; `tbwhatsapp_numero`; no gateway `POST /api/sessions` e `POST /api/sessions/{id}/start`.
- **Status/automação:** coberto por teste unitário — `OpenWaBaseUrlTests` (normalização do `BaseUrl`) e `WhatsAppNumeroServiceTests.Cadastrar_numero_inicia_a_sessao_no_gateway` (início da sessão).
### WA-12 — Listar os números sem `500`

- **Objetivo:** a tela de números carregar sem `500`, mesmo com atendimentos e mensagens no histórico.
- **Pré-condições:** pelo menos um número cadastrado; backend rodando **contra SQL Server** (`Database:UseSqlServer = true` — em InMemory o defeito **não** aparece).
- **Passos:** 1. Abrir `/whatsapp/numeros`. 2. Conferir a listagem, o contador de atendimentos ativos e a última mensagem de cada número.
- **Resultado esperado:** `GET /api/v1/whatsapp/numeros` devolve `200` com a lista completa — **nunca** "The LINQ expression 'a' could not be translated".
- **Estado atual/bloqueio:** o filtro por número é aplicado **depois** da materialização (consultas isoladas em `QueryAtendimentosAtivos`/`QueryUltimasMensagens`), porque um predicado montado em árvore de expressões não é traduzido pelo EF. ⚠️ **O provider InMemory não acusa tradução inválida**: suíte verde em InMemory **não** prova que a query roda no SQL Server — quem protege é o teste de tradução contra o **provider do SQL Server** (§19.9.2).
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `GET /api/v1/whatsapp/numeros`; `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`.
- **Status/automação:** coberto por teste unitário — `WhatsAppQueryTraducaoTests` (tradução contra o provider do SQL Server).
### WA-13 — Responder mensagem com o `BaseUrl` sem barra final

- **Objetivo:** o envio de mensagem alcançar o gateway mesmo com o `BaseUrl` configurado **sem** a barra final.
- **Pré-condições:** atendimento aberto; gateway no ar; `WhatsApp__BaseUrl` sem barra final (`http://192.168.2.253:2785/api`).
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id`. 2. Escrever e enviar a resposta. 3. Conferir o histórico e o estado da mensagem.
- **Resultado esperado:** o envio chega ao gateway (`POST /api/sessions/{id}/messages/send-text`) e a mensagem fica **enviada** — **nunca** "O gateway WhatsApp respondeu 404".
- **Estado atual/bloqueio:** o `BaseAddress` é normalizado (`NormalizarBaseUrl`), então o prefixo `/api` **não** é perdido: sem a barra, o .NET trataria `api` como nome de arquivo e **substituiria** o segmento, mandando o pedido para `/sessions` em vez de `/api/sessions` (§19.9.1).
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `POST /api/v1/whatsapp/atendimentos/{id}/mensagens`; `tbwhatsapp_mensagem`.
- **Status/automação:** coberto por teste unitário — `OpenWaBaseUrlTests` (montagem da rota com o `BaseUrl` normalizado e **sem** normalizar).
### WA-14 — Remover o número e conferir que a sessão sumiu do gateway

- **Objetivo:** a remoção do número apagar **todo o histórico dele e a sessão no gateway**, sem deixar sessão órfã nem `500` de FK.
- **Pré-condições:** perfil **Administrador**; número cadastrado, com sessão existente no gateway e **com histórico** (conversas, atendimentos, mensagens, vínculos de etiqueta, além de grupo/participantes e linhas da fila de webhook da sessão).
- **Passos:** 1. Em `/whatsapp/numeros`, abrir o menu **⋯** da linha e clicar em **Excluir número**. 2. No modal de confirmação, confirmar a primeira exclusão (o `DELETE` sai **sem** `forcar`). 3. Se o número tiver histórico, conferir a mensagem com as contagens e confirmar **"Excluir com histórico"**. 4. No gateway, listar as sessões (`GET /api/sessions`) e conferir que a sessão do número removido não está mais lá. 5. Pela API, repetir em outro número com histórico chamando `DELETE /api/v1/whatsapp/numeros/{id}` **sem** `forcar`.
- **Resultado esperado:** a tela **não** usa `confirm()` nativo: o modal próprio (`remover` → `confirmarExclusao`) envia o primeiro `DELETE` com `forcar=falso`. Número **sem** histórico recebe **`204`** na primeira confirmação. Com histórico, o backend responde **`409 { mensagem, codigo: "NUMERO_COM_HISTORICO" }`**, a `mensagem` (contagens de conversas/atendimentos/mensagens) aparece no modal e o botão vira **"Excluir com histórico"**; a segunda confirmação envia `forcar=true` e recebe **`204`**, apagando a história numa **única `SaveChanges`** (atômica) — eventos, mensagens, `tbwhatsapp_atendimento_etiqueta`, atendimentos e conversas, **mais** os **grupos**, os **participantes** desses grupos e a **fila da sessão** (`tbwhatsapp_webhook_fila.SessaoId`), que são derivados e saem **sempre**. A **definição** de `tbwhatsapp_etiqueta` sobrevive; o gateway é removido **antes** do `SaveChanges` (`DELETE /api/sessions/{id}`, com queda para o `stop`) e a falha dele **não impede** a exclusão no banco (fica no log). Pela API, **com histórico e sem `forcar`: `409`** — a regra protege só conversas/atendimentos/mensagens. Nunca mais `500` por violação de FK (grupo/participante) nem pela FK circular conversa→atendimento.
- **Estado atual/bloqueio:** correção no working tree, **ainda sem deploy** nesta revisão. A remoção de grupos/participantes/fila não depende de `forcar`; a auditoria registra `remover_forcado` quando a cascata de histórico roda. A cascata usa **entidades rastreadas e uma única `SaveChangesAsync`**, não `ExecuteDeleteAsync` — que commitaria cada statement isolado (risco de história pela metade) e não é traduzido pelo provider InMemory.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`; `DELETE /api/v1/whatsapp/numeros/{id}` (query `?forcar=`); no gateway `DELETE /api/sessions/{id}`; `tbwhatsapp_numero` e as tabelas da cascata (`tbwhatsapp_conversa`, `tbwhatsapp_atendimento`, `tbwhatsapp_mensagem`, `tbwhatsapp_evento`, `tbwhatsapp_atendimento_etiqueta`, `tbwhatsapp_grupo`, `tbwhatsapp_participante`, `tbwhatsapp_webhook_fila` — a `tbwhatsapp_etiqueta` **não** é apagada).
- **Status/automação:** coberto por teste unitário — `WhatsAppNumeroServiceTests` (**8 aprovados**: 6 existentes + 2 novos, cascata com grupo/participante/fila e número só com grupo sem conversas), incluindo `Remover_numero_exclui_a_sessao_no_gateway`, `Remover_numero_sem_rota_de_exclusao_cai_para_parar` e `Remover_forcado_com_historico_apaga_a_cascata_inclusive_a_fk_circular`.
### WA-15 — Publicar o backend e confirmar que as rotas respondem

- **Objetivo:** o backend publicado responder às rotas depois do deploy, sem `500` de corpo vazio.
- **Pré-condições:** publicação concluída (`deploy.ps1 -Publicar $true`).
- **Passos:** 1. Rodar `smoke.ps1 -Servidor <servidor-do-ambiente>`. 2. Conferir o `web.config` publicado do backend. 3. Abrir uma rota de API no navegador.
- **Resultado esperado:** o smoke passa nos quatro checks (frontend, Swagger, `runtime-config.json` e login fictício com **401**) e a API responde `200`/`401` — **nunca** `500` com corpo vazio, nem em rota inexistente (que deveria ser `404`).
- **Estado atual/bloqueio:** o `web.config` é **regerado a cada publish** e o `deploy.ps1` **não** preserva o bloco `<environmentVariables>`: ele precisa ser **reaplicado depois de cada deploy**, como **filho de `<aspNetCore>`** e com `<environmentVariable name= value=>` — o schema de `applicationHost.config` (`<add key= value=/>`) impede o ANCM de subir o processo e derruba o site inteiro. Depois de editar, recicle o pool (§13.1 e §19.9.5).
- **Rotas/endpoints/tabelas:** `web.config` do backend publicado (`C:\inetpub\wwwroot\Suporte_Back\web.config` na HML, `...\CentralOperacao\Back\web.config` na PROD); `scripts/smoke.ps1`.
- **Status/automação:** **cenário documentado**; o check **“Swagger/API”** do `smoke.ps1` é o detector do problema.
