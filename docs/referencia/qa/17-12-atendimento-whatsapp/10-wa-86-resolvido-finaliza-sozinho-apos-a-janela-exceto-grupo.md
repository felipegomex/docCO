---
id: 17-12-atendimento-whatsapp.10
titulo: "WA-86 — Resolvido finaliza sozinho após a janela, exceto grupo (Fase 2) (+3)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 2649
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-86 — Resolvido finaliza sozinho após a janela, exceto grupo (Fase 2)

- **Objetivo:** confirmar que o card parado em **Resolvido** é **finalizado e arquivado** sem operador na mesma rodada, que grupo não finaliza sozinho, que a janela é editável pelo admin e que a mensagem do cliente na janela reabre em vez de ser sobrescrita.
- **Pré-condições:** backend com `WhatsAppFinalizacaoAutomaticaService`/`Job` (registrados no `Program.cs`); perfil **Administrador** para abrir `/whatsapp/configuracao`; um atendimento privado e um de **grupo** resolvidos; operador com o Kanban aberto.
- **Passos:** 1. Abrir `/whatsapp/configuracao`, card **"Automação do atendimento"**, e salvar a janela em **1** minuto. 2. Resolver um atendimento privado. 3. Aguardar a janela (até ~2 min, respeitando o cache de 30 s do parâmetro) e conferir o Kanban. 4. Abrir o Histórico. 5. Repetir com um atendimento de **grupo**. 6. Salvar a janela em `0`, resolver outro card e aguardar: nada deve mudar. 7. Reativar a janela e, em outro card resolvido, receber mensagem do cliente **dentro** da janela. 8. Receber mensagem **depois** de um card já arquivado.
- **Resultado esperado:** o card privado **sai do Kanban operacional** e fica **Arquivado** (status 5) sozinho, com **dois** eventos na timeline — **"Finalização automática"** e **"Arquivamento"** —, ambos com `OperadorId` nulo e `UsuarioInclusao = "sistema-finalizacao"`; o primeiro traz a janela usada. O Kanban se atualiza por SignalR (`atendimento_atualizado` com `Arquivado` + `fila_alterada`) **depois** do commit, sem esperar o polling. O card de **grupo** continua em Resolvido (a regra exclui `GrupoId` e `ChatId` `@g.us`). Com a janela em `0`, a rodada não toca em nada. Mensagem dentro da janela **reabre o mesmo** atendimento; mensagem em card já arquivado abre um **novo** atendimento — o job e a reabertura usam UPDATE condicional, então um nunca sobrescreve o outro.
- **Estado atual/bloqueio:** a rodada é de **1 min** e processa até **200** cards por vez; a janela vai de **0 a 1440** minutos (`0` desliga) e é gravada por `PUT /configuracao/finalizacao-automatica` — o efeito respeita o cache de **30 s** do `WhatsAppParametroService`.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/configuracao`; `GET`/`PUT /api/v1/whatsapp/configuracao/finalizacao-automatica`; `tbwhatsapp_atendimento`, `tbwhatsapp_evento` (`finalizacao_automatica` e `arquivamento`) e `tbwhatsapp_parametro` (`atendimento.finalizacaoAutomaticaMinutos`).
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppFinalizacaoAutomaticaTests` (finaliza + arquiva + grava os dois eventos, janela, grupo, parâmetro `0` e reabertura pela mensagem) e `WhatsAppConfiguracaoTests` (GET/PUT, `400`, `403`).
### WA-87 — Foto do contato e avatar com fallback (Fase 3)

- **Objetivo:** mostrar a foto real do contato quando o gateway fornecer, sem expor a URL temporária ao navegador, e cair para iniciais (ou ícone de grupo) quando não houver imagem.
- **Pré-condições:** backend com `WhatsAppFotoContatoService` registrado (scoped); `WhatsApp_Fase8_Idempotente.sql` aplicado (`WAT_FotoUrl`/`WAT_FotoAtualizadaEm`); número pareado no OpenWA; contato com conversa (a sessão é resolvida pela conversa mais recente); foto disponível no WhatsApp e um contato sem foto.
- **Passos:** 1. Abrir `/whatsapp` e conferir os avatares do card. 2. Abrir `/whatsapp/atendimento/:id` e conferir o avatar no cabeçalho da conversa, no cabeçalho do atendimento e no painel do cliente. 3. Abrir `/whatsapp/minhas-conversas`. 4. Abrir um contato **sem foto**. 5. Abrir um atendimento de **grupo**. 6. Inspecionar a resposta de `GET /api/v1/whatsapp/contatos/{id}/foto`. 7. Repetir a abertura de telas e conferir no log/Network que a foto não é rebaixada a cada troca.
- **Resultado esperado:** a foto aparece no avatar quando o gateway devolve URL acessível; a chamada é `GET /api/v1/whatsapp/contatos/{id}/foto` (blob autenticado), **nunca** a URL do gateway. Resposta `200` com `image/*` (SVG recusado), limite de **2 MB**, `X-Content-Type-Options: nosniff` e `Cache-Control: private, max-age=600`. Sem foto/indisponível, a rota responde **`404`** e o avatar cai nas **iniciais**; grupo mostra `bi-people` sem buscar imagem. A URL fica em cache por `atendimento.fotoTtlHoras` (padrão **24 h**), com cache negativo quando não há foto, uma nova consulta se a URL expirar e cache em memória de **10 min**; o frontend reaproveita o blob por contato (`shareReplay`) e revoga o object URL ao destruir.
- **Estado atual/bloqueio:** falha do gateway (`404`, `429`, timeout) **não** vira erro na tela — `404` entra no cache negativo e erros de rede/gateway são logados com retorno `null`; content-type inválido ou imagem acima do limite são descartados. Se o contato não tem conversa nem URL antiga válida, não há como obter a foto. Sem teste automatizado dedicado.
- **Rotas/endpoints/tabelas:** `/whatsapp`, `/whatsapp/minhas-conversas`, `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/contatos/{id}/foto`; gateway `GET /api/sessions/{sessao}/contacts/{chatId}/profile-picture`; `tbwhatsapp_contato` (`WAT_FotoUrl`, `WAT_FotoAtualizadaEm`), `tbwhatsapp_parametro` (`atendimento.fotoTtlHoras`).
- **Status/automação:** **cenário derivado do código**; backend coberto por `WhatsAppFotoContatoTests` (3 casos: download/persistência, cache negativo e falha do gateway) e avatar por `contato-avatar.component.spec.ts` (3 casos, **não executado**).
### WA-88 — Métricas por etiqueta e filtro de etiqueta (Fase 4)

- **Objetivo:** medir volume, tempo médio de resolução e reabertura por etiqueta, e recortar os conjuntos de **abertos/resolvidos** do painel por uma etiqueta — deixando explícito que a soma por etiqueta não fecha com o total.
- **Pré-condições:** catálogo de etiquetas com pelo menos uma ativa e uma inativa com vínculo histórico; atendimentos no período com **0, 1 e várias** etiquetas; pelo menos um atendimento resolvido e um reaberto.
- **Passos:** 1. Abrir `/whatsapp/metricas` (aba **Geral**). 2. No filtro de etiqueta, escolher “Todas as etiquetas” e conferir o bloco **“Atendimentos por etiqueta”**. 3. Selecionar uma etiqueta ativa e aplicar. 4. Selecionar uma etiqueta **inativa** com vínculo e aplicar. 5. Comparar a soma das quantidades com o cartão **Recebidos**.
- **Resultado esperado:** `GET /api/v1/whatsapp/metricas` aceita `etiquetaId` e devolve `porEtiqueta[]` com `etiquetaId`, `nome`, `cor`, `ativo`, `quantidade`, `tempoMedioResolucaoMinutos` e `taxaReabertura`. A quantidade é de atendimentos **distintos**; o tempo médio usa somente os atendimentos abertos no período que já possuem resolução e **desconta a espera do cliente**; a reabertura é o percentual dos atendimentos da etiqueta com `ContadorReaberturas > 0`. Etiquetas inativas aparecem com `ativo=false` e o rótulo “inativa”. O bloco mostra chip na cor da etiqueta, quantidade, tempo médio e reabertura (`toFixed(1)%`), e o estado vazio “Nenhum atendimento com etiqueta no período selecionado.”. O aviso fixo explica que **um atendimento pode ter várias etiquetas e a soma não fecha com o total geral**. Com `etiquetaId`, os conjuntos de abertos e resolvidos são recortados pelo vínculo; o filtro também recarrega a lista de destaques.
- **Estado atual/bloqueio:** a **soma por etiqueta não é** um total alternativo — é uma contagem por vínculo. O filtro por etiqueta **não** é aplicado por `metricas/qualidade` hoje (`CalcularQualidadeAsync` só considera `numeroId` e `responsavelId`), embora o frontend envie o parâmetro. `porEtiqueta` tem default `null` no contrato; o serviço devolve lista vazia quando não há vínculo.
- **Rotas/endpoints/tabelas:** `/whatsapp/metricas`; `GET /api/v1/whatsapp/metricas?etiquetaId=`; `tbwhatsapp_atendimento`, `tbwhatsapp_atendimento_etiqueta`, `tbwhatsapp_etiqueta`, `tbwhatsapp_evento`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppMetricasEtiquetaTests` (2 casos: volume/tempo/reabertura por etiqueta e lista vazia).
### WA-89 — Cadastro/edição de número exige setor ativo, e o Kanban continua filtrando por setor

- **Objetivo:** garantir que todo número pertença a exatamente um setor ativo da `tbfuncao` e que a troca do texto livre por FK não quebre o filtro por fila do Kanban.
- **Pré-condições:** perfil **Administrador**; `tbfuncao` com pelo menos um setor ativo e um inativo; pelo menos um número já cadastrado (backfill do setor aplicado).
- **Passos:** 1. Abrir `/whatsapp/numeros` e clicar em **Novo número**. 2. Conferir que o select **Setor** lista somente setores ativos. 3. Conferir que **Salvar** fica desabilitado sem descrição/setor e habilitado quando o formulário é válido. 4. Salvar com um setor ativo. 5. Editar o número e trocar o setor. 6. Pela API, enviar `funcaoId` de setor **inativo** e de setor **inexistente**. 7. Abrir `/whatsapp` e filtrar a fila pelo nome do setor.
- **Resultado esperado:** no modal, **Descrição** e **Setor** são obrigatórios e o botão **Salvar** só habilita com `formValido()` (descrição não vazia e `funcaoId` preenchido); `salvandoForm()` mantém a validação como segunda barreira ("Informe a descricao do numero." / "Selecione o setor responsavel pelo numero."). O backend devolve `400 { mensagem }` para setor ausente, inexistente ou inativo; o `POST` devolve `201` e o `PATCH` `200` com `funcaoId` preenchido e `fila` igual a `tbfuncao.DESCRICAO`. A coluna **Setor** da tabela mostra esse nome; `fila` não existe como campo de entrada. O filtro `fila` de `GET /atendimentos` continua funcionando porque compara `Numero.Funcao.Descricao` — os cards do número aparecem sob o novo nome depois da troca.
- **Estado atual/bloqueio:** exige o schema consolidado (`WAT_FuncaoId NOT NULL`, `WAT_Fila` dropada). Antes da consolidação, o código novo funciona com a coluna anulável e o backfill; o `preflight-schema.ps1` bloqueia o publish sem `WAT_FuncaoId` e a FK. Números criados pelo código antigo durante a janela recebem o fallback no script de consolidação.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`, `/whatsapp`; `POST/PATCH /api/v1/whatsapp/numeros`, `GET /api/v1/whatsapp/atendimentos?fila=`; `tbwhatsapp_numero` (`WAT_FuncaoId`), `tbfuncao`; scripts `WhatsApp_NumeroFuncao_Idempotente.sql` e `WhatsApp_NumeroFuncao_Consolidar_Idempotente.sql`.
- **Status/automação:** **cenário derivado do código**; coberto por `WhatsAppNumeroServiceTests` (`Cadastrar_numero_sem_setor_e_rejeitado`, `Cadastrar_numero_com_setor_inexistente_e_rejeitado`, `Cadastrar_numero_com_setor_inativo_e_rejeitado` e `Editar_numero_troca_o_setor_e_o_toggle_de_ativo`).
