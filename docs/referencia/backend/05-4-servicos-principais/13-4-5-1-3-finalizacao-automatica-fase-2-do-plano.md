---
id: 05-4-servicos-principais.13
titulo: "4.5.1.3 Finalização automática (Fase 2 do plano)"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-11-01"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 659
ficha: docs/modulos/README.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

#### 4.5.1.3 Finalização automática (Fase 2 do plano)

- **Job e service separados.** `WhatsAppFinalizacaoAutomaticaJob` (`BackgroundService`) só mantém o `PeriodicTimer` de **1 min** e cria um escopo de DI por rodada; a regra está em `WhatsAppFinalizacaoAutomaticaService` (Scoped), registrado em `Program.cs` junto do hosted service. Erro em uma rodada é logado e o loop continua.
- **Seleção e escrita condicional.** A rodada lê até **200** candidatos (`Resolvido` com `ResolvidoEm <= corte`, conversa sem grupo) ordenados por `ResolvidoEm` e chama `ArquivarCondicionalAsync` (o antigo `FinalizarCondicionalAsync`) por id com o **mesmo** filtro de status/corte; o `ExecuteUpdateAsync` grava **`Arquivado`** e `UsuarioAlteracao = "sistema-finalizacao"`. Os eventos `finalizacao_automatica` **e** `arquivamento` (`OperadorId = null`) só são gravados quando o update afeta **1 linha** — quem chegar primeiro entre o job e a reabertura vence, e o outro não sobrescreve nada. O InMemory (testes) revalida o estado na leitura, porque não executa UPDATE em massa. O SignalR publica `Arquivado` (`atendimento_atualizado` + `fila_alterada`) depois do commit, em best-effort.
- **Janela configurável.** A janela em minutos vem de `atendimento.finalizacaoAutomaticaMinutos` (padrão **15**; `<= 0` desliga) e pode ser lida/gravada pelo administrador em `GET`/`PUT /api/v1/whatsapp/configuracao/finalizacao-automatica` (`FinalizacaoAutomaticaConfig`, `0..1440`), que usa `IWhatsAppParametroService` e o operador do token. A alteração respeita o cache de **30 s** do parâmetro.
- **Sem schema e sem migration.** A fase usa apenas `tbwhatsapp_atendimento`, `tbwhatsapp_evento` e o parâmetro já criado por `WhatsApp_Fase8_Idempotente.sql`; não há script novo.

**Cobertura da Fase 6 (2026-10-27).** As quatro fases do plano ganharam suíte própria, todas verdes: [`WhatsAppProtocoloTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppProtocoloTests.cs) (**3** — projeção de protocolo/telefone na lista, busca por protocolo e id, e filtro de protocolo exato sem prefixo), [`WhatsAppFinalizacaoAutomaticaTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppFinalizacaoAutomaticaTests.cs) (finaliza + arquiva + grava os dois eventos, além da janela, grupo, parâmetro `0` e reabertura pela mensagem), [`WhatsAppFotoContatoTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppFotoContatoTests.cs) (**3** — download com persistência de URL/data, cache negativo sem foto e falha do gateway sem erro para a tela) e [`WhatsAppMetricasEtiquetaTests.cs`](../../../../backend/Central_BackEnd.Tests/WhatsAppMetricasEtiquetaTests.cs) (**2** — volume/tempo/reabertura por etiqueta e lista vazia). O projeto fechou em **921 casos**; o frontend ganhou `contato-avatar.component.spec.ts` (**3** — iniciais, grupo e proxy da foto) e 1 caso de busca por protocolo/telefone em `whatsapp.utils.spec.ts`, sem execução do Karma nesta revisão.

