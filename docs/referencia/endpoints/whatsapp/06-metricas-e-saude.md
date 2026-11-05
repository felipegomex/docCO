---
id: whatsapp.06
titulo: "Métricas e saúde"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 1207
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Métricas e saúde

| Método | Rota | Entrada | Resposta resumida |
|---|---|---|---|
| `GET` | `/metricas` | Query `MetricasFiltro`: `periodoDe?` (padrão: 30 dias), `periodoAte?`, `numeroId?`, `responsavelId?`, **`etiquetaId?`** (Fase 4) | `200 MetricasWhatsApp`: total recebido, em andamento, resolvidos, reaberturas, transferências, falhas de envio, média e mediana do tempo até a 1ª resposta e de resolução (**em minutos**, já **descontando a espera do cliente**), fila de não atendidos, idade do mais antigo, contagem por status, por operador, por número e por dia, **e `porEtiqueta[]`** (Fase 4: `etiquetaId`, `nome`, `cor`, `ativo`, `quantidade`, `tempoMedioResolucaoMinutos`, `taxaReabertura`). `etiquetaId` recorta **abertos e resolvidos** pelo vínculo (`EXISTS`). `400` se o fim do período for anterior ao início |
| `GET` | `/saude` | — | `200 List<SaudeNumeroWhatsApp>`: espelho da Central **e** o que o gateway responde agora (`statusGateway`, `ultimoErro`, `gatewayAcessivel`), mais a última mensagem |
| `GET` | `/metricas/qualidade` | Query `MetricasFiltro` | `200 MetricasQualidadeWhatsApp`: taxa de reabertura, FCR e transferências, agrupados **por cliente**. **Só administrador** (`403` para os demais); `400` período invertido. **Por categoria** foi removido junto com a Fase D. O filtro compartilhado aceita `etiquetaId`, mas `CalcularQualidadeAsync` hoje aplica apenas `numeroId` e `responsavelId` — o recorte por etiqueta vale para `/metricas`, não para esta rota |

O "em andamento" é o estado no **fim** do período e inclui o status `2` (Aguardando cliente). O status `6` (Visto) **não** conta como resolvido nem como atendimento ativo. O tempo em "Aguardando cliente" é **descontado** das duas durações: a espera é do cliente, não atraso do suporte.

**FCR (rodada 2026-11-01).** Em `CalcularQualidadeAsync`, a consulta `resolvidosQuery` considera resolvidos os status **`Resolvido` (3), `Finalizado` (4) e `Arquivado` (5)**. O FCR é `atendimentos resolvidos sem reabertura / total de resolvidos`; como a finalização automática leva `Resolvido` direto para `Arquivado`, contar apenas o status 3 zeraria o denominador e faria o atendimento "desaparecer" da qualidade.

**Fase 4 — métricas por etiqueta.** `porEtiqueta` é calculado sobre os atendimentos **abertos no período** (com os mesmos filtros de número, responsável e etiqueta). Cada item conta atendimentos **distintos** vinculados à etiqueta; `tempoMedioResolucaoMinutos` considera somente os atendimentos **abertos no período que já possuem resolução** e desconta a espera do cliente; `taxaReabertura` é o percentual dos atendimentos da etiqueta com `ContadorReaberturas > 0` (0–100). Etiquetas **inativas** aparecem quando têm vínculo histórico (`ativo=false`). A lista é ordenada por quantidade (desc) e nome. Como um atendimento pode ter **várias** etiquetas, a **soma da lista não fecha com o total geral** — a tela exibe esse aviso. O contrato declara `porEtiqueta` com default `null`; o serviço devolve a lista (vazia quando não há vínculo).

O resumo de cada atendimento (`AtendimentoResumo`, usado nas listas) carrega também `SlaMinutos`, `SlaMetaMinutos` e `SlaSituacao` — a **situação geral** do SLA (`no_prazo`, `critico`, `atrasado`, `estourado`; a Fase 05 do novo ciclo acrescentou o `critico`), calculada na leitura pela regra vigente (com o percentual de aviso configurado) e igual ao **pior caso** entre primeira resposta e resolução. O detalhe (`AtendimentoDetalhe`) traz ainda `SlaOperacao` com as duas dimensões **independentes** (`PrimeiraResposta` e `Resolucao`, cada uma com minutos, meta e situação) e os valores vigentes para o card (`MinutosVigentes`/`MetaVigente`). Desde a **Fase 1 do plano**, o mesmo resumo traz `protocolo` e `contatoTelefone` já **formatado pelo backend** (a tela não recalcula).

**Fase 07 (Kanban Service Desk):** o resumo passou a trazer também `empresaId`/`empresaNome` (mesmo valor de `contatoEmpresa*`, mantido por compatibilidade), `etiquetas[]`, `emInvestigacao`, `primeiraRespostaSlaSituacao` e `resolucaoSlaSituacao` — as duas situações **independentes**, enquanto `slaSituacao` continua sendo a geral (pior caso). As etiquetas da página inteira são carregadas em **uma consulta agrupada** (sem N+1) e o filtro por etiqueta vira `EXISTS`; a projeção do card não carrega coleções. `emInvestigacao` é **indicador**, nunca status/coluna.

**Fase 1 do plano (protocolo e telefone):** o `protocolo` é uma coluna **computada PERSISTED** (`WA-AAAAMMDD-000123`, derivada de `WAT_Id` + `WAT_DataInclusao`) com índice único; o filtro `protocolo` é **igualdade exata** e a busca livre também o cobre. No **detalhe**, o telefone de exibição vem de `telefoneResolvido ?? telefoneNormalizado ?? numero`; na **lista**, o card usa a chave normalizada (`contatoTelefone`). Nos dois casos o backend formata. Em contato `@lid`, o telefone resolvido é **apenas exibição**, nunca chave de identidade.

