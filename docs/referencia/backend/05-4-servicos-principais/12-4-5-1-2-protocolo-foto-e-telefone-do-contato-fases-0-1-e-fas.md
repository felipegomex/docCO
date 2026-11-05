---
id: 05-4-servicos-principais.12
titulo: "4.5.1.2 Protocolo, foto e telefone do contato (Fases 0/1 e Fase 3 do plano)"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 597
ficha: docs/modulos/README.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

#### 4.5.1.2 Protocolo, foto e telefone do contato (Fases 0/1 e Fase 3 do plano)

- **Protocolo computado.** `Atendimento.Protocolo` é mapeado com `HasComputedColumnSql(..., stored: true)` e índice único; a expressão é a mesma do script `WhatsApp_Fase8_Idempotente.sql` (`WA-` + `WAT_DataInclusao` em `AAAAMMDD` + id com 6 dígitos). O EF **nunca** escreve a coluna. O setter público existe só para o provider InMemory dos testes.
- **Telefone e foto.** `Contato` ganhou `FotoUrl`, `FotoAtualizadaEm` e `TelefoneResolvido`; no **detalhe**, o `AtendimentoService` monta `ContatoTelefone` com `TelefoneResolvido ?? TelefoneNormalizado ?? Numero`; na **projeção da lista**, o card usa `TelefoneNormalizado`. Ambos são formatados por `TelefoneNormalizador.Formatar` (o `@lid` usa o resolvido apenas como display). Os DTOs de contato (`ContatoResumoDto`, `ContatoFichaDto`, `ContatoAdminDto` e `ContatoContextoDto`) expõem `TelefoneExibicao`/`TemFoto`/`FotoAtualizadaEm`, e as projeções **preenchem** os três (`temFoto` deriva de `FotoUrl` não vazio). A rota `GET /contatos/{id}/foto` serve a imagem por proxy autenticado — a URL temporária do gateway nunca vai ao navegador.
- **Entrada e novos chats.** `RegistrarEntradaAsync` grava `WAT_Numero` com o telefone **normalizado e sem JID**, mantendo `ChatId` como o JID usado no envio; para `@lid` (sem telefone normalizado) tenta `ResolverTelefoneAsync` e grava `WAT_TelefoneResolvido` só como display. `SolicitarPrivadoAsync` monta o `ChatId` com `TelefoneNormalizador.NormalizarTelefone` (`digitos@c.us`) e usa o `ChatId` do contato na conversa.
- **Parâmetros.** `atendimento.finalizacaoAutomaticaMinutos` (15, `0` desliga) e `atendimento.fotoTtlHoras` (24) estão cadastrados e expostos como constantes em `WhatsAppParametros`; `TiposEventoWhatsApp.FinalizacaoAutomatica` (`finalizacao_automatica`) está definido. Desde a **Fase 2**, a janela de finalização é **consumida pelo job** (§4.5.1.3); o `fotoTtlHoras` é consumido pelo `WhatsAppFotoContatoService` como TTL da URL e do cache negativo.

