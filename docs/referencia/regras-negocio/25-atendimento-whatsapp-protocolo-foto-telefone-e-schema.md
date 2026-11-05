---
id: regras-negocio.25
titulo: "Atendimento WhatsApp — protocolo, foto e telefone do contato e schema"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, whatsapp, protocolo, telefone, foto, schema]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1000
ficha: docs/modulos/whatsapp.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).
> Detalhe extraído de [16 — atribuição, ciclo de vida e "Aguardando cliente"](./16-modulo-de-atendimento-whatsapp-atribuicao-ciclo-de-vida-e-ag.md).

## Protocolo, foto e telefone do contato (Fases 0/1, 2 e 3 do plano — 2026-10-27)

**Situação: protocolo e telefone em uso na leitura; finalização automática em operação; foto implementada com proxy autenticado e fallback para iniciais.**

- **Protocolo do atendimento.** `WAT_Protocolo` é uma coluna **computada PERSISTED** no SQL Server: `WA-AAAAMMDD-000123`, derivada de `WAT_Id` (`000123`) e `WAT_DataInclusao` (`AAAAMMDD`). É estável porque nenhum dos dois campos muda; não há `SEQUENCE` (inviável no compatibility 100) nem segunda gravação no código. Há **índice único**, e o valor é **somente leitura** (o EF nunca o escreve).
- **Busca e exibição.** O filtro `protocolo` é **igualdade exata** depois de `trim` + maiúsculas; a busca livre também encontra o protocolo, o **id**, a empresa e o telefone (a partir de 4 dígitos). Na tela, `protocoloDe` mostra o protocolo e cai em `#<id>` quando a resposta não traz o campo — nunca mostra vazio nem "null". O protocolo aparece no card do Kanban, em Minhas Conversas, no cabeçalho e nos dados do atendimento.
- **Telefone do contato.** O **detalhe** expõe `contatoTelefone` e os DTOs de contato expõem `telefoneExibicao`, formatados pelo backend com `TelefoneResolvido ?? TelefoneNormalizado ?? Numero` (a listagem administrativa formata o `Numero`); a **lista** do Kanban usa a chave normalizada, também formatada. Para `@lid`, `WAT_TelefoneResolvido` é **display**, não chave de identidade: a duplicidade e a unificação continuam usando a chave normalizada (e o `@lid` segue fora dela). Na entrada, `WAT_Numero` é gravado **normalizado e sem JID** (o `ChatId` continua sendo o JID); para `@lid`, o gateway pode resolver o telefone, que fica só em `WAT_TelefoneResolvido`. `SolicitarPrivadoAsync` monta o `ChatId` com o normalizador (`digitos@c.us`).
- **Foto do contato.** A URL temporária do OpenWA fica em `WAT_FotoUrl` e **nunca** é enviada ao navegador: o backend baixa a imagem e a serve por `GET /api/v1/whatsapp/contatos/{id}/foto`, com `nosniff` e `Cache-Control: private, max-age=600`. O TTL da URL/cache negativo é `atendimento.fotoTtlHoras` (**24 h** padrão): dentro da janela, a URL é reaproveitada; fora dela, o gateway é consultado e a data é gravada **mesmo quando não há foto** (cache negativo), para não martelar a rota. O download aceita apenas `image/*` (SVG recusado), com limite de **2 MB**, e faz **uma** nova tentativa se a URL expirou. Há cache em memória de **10 min**; falha do gateway vira `404`/iniciais, nunca erro na tela.
- **Parâmetros.** Ficam em `tbwhatsapp_parametro`: `atendimento.finalizacaoAutomaticaMinutos` = **15** (0 desliga) e `atendimento.fotoTtlHoras` = **24**. O tipo de evento `finalizacao_automatica` está definido e, desde a **Fase 2**, a janela de finalização é **consumida pelo job** (regra na tabela de regras de atendimento); desde 2026-11-01 a janela também é editável pelo admin em `GET`/`PUT /whatsapp/configuracao/finalizacao-automatica`. Desde a **Fase 3**, o `fotoTtlHoras` é consumido pelo `WhatsAppFotoContatoService`.
- **Schema.** Script aditivo e idempotente `WhatsApp_Fase8_Idempotente.sql`, que também faz o **backfill de `WAT_Numero` sem o JID** (`@c.us`). Rodar no banco do ambiente antes do publish; sem ele, a leitura do atendimento falha por coluna inexistente.
