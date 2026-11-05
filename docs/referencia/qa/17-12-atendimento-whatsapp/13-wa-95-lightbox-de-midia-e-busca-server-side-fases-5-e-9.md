---
id: 17-12-atendimento-whatsapp.13
titulo: "WA-95 — Lightbox de mídia e busca server-side (Fases 5 e 9 do plano 2026-10) (+1)"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - frontend/src/app/features/whatsapp/components/midia-mensagem/midia-mensagem.component.spec.ts
  - frontend/src/app/features/whatsapp/pages/minhas-conversas/minhas-conversas-busca.component.spec.ts
  - backend/Central_BackEnd.Tests/WhatsAppContatoTests.cs
tokens_estimados: 1300
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-95 — Lightbox de mídia na própria tela (Fase 5 do plano 2026-10)

- **Objetivo (H.4):** confirmar que imagem e vídeo abrem em **lightbox** (modal na própria tela) sobre o MESMO blob autenticado, com proporção preservada, download do blob e "Tamanho original" — sem nenhuma URL pública de mídia.
- **Pré-condições:** conversa com imagem recebida (ex.: 1920×1080), vídeo, áudio e documento; uma mídia já removida pela retenção (responde `404`).
- **Passos:** 1. Clicar o botão "Ampliar" no balão de uma imagem. 2. Conferir no Network: nenhuma requisição nova — o modal usa o blob já baixado de `GET .../mensagens/{id}/midia`. 3. Clicar "Tamanho original": conferir o rótulo com as dimensões naturais e a alternância ajustar-à-tela ↔ escala 1:1 com scroll interno. 4. Clicar "Baixar" e conferir o arquivo com o nome original. 5. Abrir um **vídeo** no mesmo viewer, pressionar Esc e depois clicar no fundo. 6. Conferir que áudio e documento **não** entram no viewer. 7. Abrir mídia retida (404).
- **Resultado esperado:** `abrirLightbox()` abre o modal sobre o blob autenticado — **nenhuma chamada a `window.open`** (o método `abrirEmNovaAba()` foi removido). Proporção sempre preservada (`object-fit: contain`); os limites do balão saíram do estilo inline para SCSS responsivo (máx. 280px padrão; 200px abaixo de 576px; imagem 340px a partir de 1400px). `registrarDimensoes(el)` capta `naturalWidth/naturalHeight` (imagem) ou `videoWidth/videoHeight` (vídeo) em `dimensoesOriginais`, e `rotuloTamanhoOriginal()` os exibe no botão. `aoFecharLightbox()` (Esc ou clique no fundo) pausa o vídeo, fecha via `fecharLightbox()` e **não revoga o blob** ainda em uso. Download usa o blob + nome original. Áudio/documento ficam fora do viewer (decisão registrada no plano § 9). O 404 segue avisando "pode ter sido removido pela política de retenção".
- **Estado atual/bloqueio:** frontend puro; contrato inalterado — mídia só sai por `GET .../mensagens/{id}/midia`, autenticado. Sem mudança de backend, schema ou permissão.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas/:id`; `GET /api/v1/whatsapp/mensagens/{id}/midia`; nenhum endpoint novo; nenhuma tabela nova.
- **Status/automação:** **cenário derivado do código**; coberto por `frontend/src/app/features/whatsapp/components/midia-mensagem/midia-mensagem.component.spec.ts` (7 casos: só blob autenticado; clique abre o modal com o mesmo blob e **não** chama `window.open`; "Tamanho original" mostra 1920×1080 e alterna o modo; download usa blob+nome; vídeo no mesmo viewer com Esc fechando e pausando; clique no fundo fecha sem revogar o blob; regressão do 404 de retenção).

### WA-96 — Busca server-side e paginação incremental (Fase 9 do plano 2026-10)

- **Objetivo (H.6):** confirmar que Minhas Conversas localiza atendimentos **fora dos 100 recentes** por nome, telefone, protocolo e status, que o mini-form de contato do composer encontra contato pelo **número**, e que não há regressão no Kanban; medir lista grande (< 2 s).
- **Pré-condições:** operador com mais de 100 atendimentos recentes (busca alcançável só pelo servidor); contatos com nome, número e `TelefoneNormalizado`; DevTools com Network aberto.
- **Passos:** 1. Em `/whatsapp/minhas-conversas`, digitar um nome na busca sem pausar a digitação: conferir **uma** requisição após ~300 ms com `busca` na query de `GET /atendimentos`. 2. Repetir com telefone (≥ 4 dígitos), protocolo e `Id`. 3. Alternar o status pelos **chips** (Abertas/Todas/status 0–6/Arquivado) e conferir o recorte server-side. 4. Clicar "Carregar mais" e conferir a próxima página de 50 anexada (sinal `temMais`). 5. `Ctrl+K` ainda foca a busca. 6. No composer do atendimento, abrir o mini-form de contato e digitar ≥ 2 caracteres (inclusive só dígitos de número), conferindo a busca **server-side**. 7. Abrir o Kanban e refazer a busca lá. 8. Medir o tempo de uma lista grande.
- **Resultado esperado:** busca de Minhas Conversas é **server-side** com debounce de 300 ms (`busca$`, `aoDigitarBusca()`, `buscaAplicada`; `DEBONCE_BUSCA_MS = 300`) — **não refiltra no cliente** (`filtrarMinhasConversas` saiu de `whatsapp.utils.ts` com seu bloco de spec); o teto de 100 virou paginação incremental (`TAMANHO_PAGINA = 50`, `carregarMais()`, signals `pagina`/`carregandoMais`, computed `temMais`, no padrão `irParaPagina` do Kanban), com teto global de **200** por página na API (`PaginadoDefaults` — item além do teto não é alcançável, limitação documentada). O filtro de status virou **chips** (`.wa-mc__chips`/`.wa-mc__chip`) no lugar do `<select>`, mantendo `montarFiltro` server-side; ordenação e `Alt+1..9` permanecem locais sobre a lista carregada. O mini-form de contato do composer usa `listarContatos(busca)` — **fonte única**, mínimo 2 caracteres (`MINIMO_BUSCA`). O `busca` de `GET /contatos` agora cobre **Nome OR ChatId OR Numero OR TelefoneNormalizado**, alinhado ao predicate do admin. Kanban inalterado (mesmo param `busca`, tela própria).
- **Estado atual/bloqueio:** diagnóstico 9a registrado nas observações de contrato do WhatsApp (`/atendimentos?busca` já era server-side paginada; `LIKE '%x%'` com curinga à esquerda **não é sargable**). **Índices não avaliados nesta sessão** (ponte MCP sql-server indisponível) e **nenhum índice criado** — regra: só com lentidão provada + SQL idempotente + autorização explícita. **Pendência humana:** medir lista grande (< 2 s) e decidir índices se necessário.
- **Rotas/endpoints/tabelas:** `/whatsapp/minhas-conversas`; `GET /api/v1/whatsapp/atendimentos`, `GET /api/v1/whatsapp/contatos`; nenhuma tabela nova; nenhum índice criado.
- **Status/automação:** **cenário derivado do código**; coberto por `frontend/src/app/features/whatsapp/pages/minhas-conversas/minhas-conversas-busca.component.spec.ts` (debounce, uma requisição por termo, carregar mais) e pelo backend `backend/Central_BackEnd.Tests/WhatsAppContatoTests.cs` (`ListarAsync_busca_alcanca_o_numero_alinhado_ao_predicate_do_admin`); o mini-form do composer é coberto por `whatsapp-composer.component.spec.ts`.
