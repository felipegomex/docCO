---
id: 05-4-servicos-principais.11
titulo: "4.5.1.1 Filtros e projeção do Kanban (Fase 07)"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 960
ficha: docs/modulos/kanban.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

#### 4.5.1.1 Filtros e projeção do Kanban (Fase 07)

`AtendimentoService.ListarAsync` passou a devolver o **recorte** do Kanban, não só a página:

| Etapa | Implementação confirmada |
|---|---|
| `AplicarFiltros` (estático, exposto ao projeto de testes) | Traduz para SQL status, **`somenteAbertos`** (`status` 0, 1 e 2 em `OR`; combinado com `status`, vale a interseção), número, fila, período, responsável, `somenteMeus`, **prioridade**, **empresa do contato** (navegação `Conversa.Contato.EmpresaId`), **etiqueta** (`Atendimento.Etiquetas.Any(...)` → `EXISTS`), **investigação** e busca (assunto/`chatId`/nome do contato). Desde as Fases 0/1 do plano, `protocolo` é **igualdade exata** (normalizada para maiúsculas) na coluna computada indexada, e a busca livre também cobre **protocolo**, **id**, empresa e telefone (a partir de 4 dígitos). `semResponsavel` continua exclusivo com `responsavelId` (400) |
| `AplicarIds` com `EF.Constant` | Restringe aos ids calculados pelo SLA com lista **literal** (`IN (1, 2, 3)`), sem o `OPENJSON` que o EF 8 geraria e que o banco legado (compatibility 100) não aceita |
| `FiltrarPorSlaAsync` | O SLA depende de expediente/espera e não é traduzível em SQL: busca só os ids candidatos, classifica em memória pelo **mesmo** `IWhatsAppSlaService` do painel e aplica os ids **antes** de `CountAsync`/`Skip`/`Take`. Sem filtro de SLA, a paginação continua no banco |
| `ValidarSituacaoSla` | Aceita apenas `no_prazo`, `critico`, `atrasado` e `estourado`; qualquer outro valor → `ArgumentException` → **400** (não lista vazia silenciosa) |
| Projeção do card | Uma consulta para a página, com empresa já por navegação; `IdadeMinutos` e rótulo do tipo de contato entram depois (dependem de “agora”/método customizado). O SLA é calculado em consulta própria e enriquece `SlaMinutos`, `SlaMetaMinutos`, `SlaSituacao`, `PrimeiraRespostaSlaSituacao` e `ResolucaoSlaSituacao` |
| `CarregarEtiquetasAsync` + `QueryEtiquetas` (estáticas) | **Uma** consulta agrupada para todos os cards da página, com `EF.Constant` na lista de atendimentos e `orderby etiqueta.Nome`; não filtra `Ativo` (o vínculo histórico continua visível). **Sem N+1** |
| Enriquecimento | `EmpresaId`/`EmpresaNome` repetem `ContatoEmpresa*` (compatibilidade); `EmInvestigacao` é lido da coluna; o detalhe usa o mesmo enriquecimento |

A Fase 07 **não altera o schema**: a navegação `Atendimento.Etiquetas` (`ICollection<AtendimentoEtiqueta>`) já existe desde a Fase 06 e passa a ser usada como leitura. Cobertura: 15 casos em [`WhatsAppKanbanServiceDeskTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppKanbanServiceDeskTests.cs) (filtros, total/paginação, contexto do card, consulta agrupada e tradução SQL sem `OPENJSON`).

A Fase 08 também **não altera o schema**: `ContatoAdminService` só lê `tbwhatsapp_contato`, `tbwhatsapp_conversa`, `tbwhatsapp_atendimento` e `tbcliente`. A agregação da lista (`TotalAtendimentos`/`UltimoAtendimentoEm`/`UltimoAtendimentoId`) é feita por **subconsultas correlacionadas na mesma query paginada** (sem N+1) e a consulta é exposta para o teste de tradução SQL. Cobertura: 17 casos em [`WhatsAppAdminContatoTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppAdminContatoTests.cs) (filtros isolados e combinados, paginação funcional com conferência da **segunda página**, agregações, `@lid`, histórico, `403` sem perfil, tradução SQL sem `OPENJSON` e um caso específico de ordenação/paginação após a projeção escalar, exigindo `ORDER BY`/`OFFSET`/`FETCH NEXT`).

