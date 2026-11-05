# Plano Jota 3D — mascote articulado do chat (Caminho B)

> Status: **aprovado pelo usuário em 21/10/2026**. Execução por fases; cada fase fecha com `scripts/validate.ps1` verde.
> Escopo v1 aprovado: **`/chat` (herói + estado vazio) + widget JOTA (painel aberto)**.
> Pipeline do modelo aprovado: **geração 3D por IA (Meshy/Tripo/Rodin) → auto‑rig Mixamo → clipes/export no Blender → `.glb`**.

## Contexto

O JOTA é o assistente de IA da Central (persona por setor, catálogo `Chat:Personas`). Este plano adiciona o mascote **3D de verdade** (malha articulada + clipes animados) ligado aos **estados reais** de processamento — sem que a animação determine o resultado da operação.

Referências visuais no repositório:

- `frontend/src/assets/images/jota.png` (1024×866, 537 KB) — render principal; fallback v1.
- `frontend/src/assets/images/JOTA/JOTA-1.png` — prancha de 6 poses (referência do modelo):
  idle = braços cruzados; thinking = mão no queixo; processing = correndo; loading = no laptop;
  success = acenando; error = cara triste + joinha.

Restrições verificadas na auditoria (21/10/2026):

- Angular 18.2, **CSR puro** (sem SSR), zone.js ativo, budget inicial **já estourado** (1,06 MB / 1 MB warn) → `three` **nunca** pode entrar no bundle inicial.
- `web.config` do IIS sem `mimeMap` para `.glb` → precisa da fase 4 antes do smoke.
- Sem serviço global de loading: estados vêm dos **signals locais** das telas (`carregando`, `enviando`, `analisandoIa`).
- Backend **não muda**: nenhuma animação justifica endpoint.

## Divisão de responsabilidade

| Quem | O que |
|---|---|
| Agente (código) | Componente 3D, viewer three.js, estados, fallback, integrações, testes, `web.config`, docs, validação |
| Externo (usuário/artista) | Produzir `jota.glb` articulado pelo pipeline aprovado; o agente **não** modela personagens |

## Contrato do modelo 3D (para o pipeline externo)

| Item | Exigência |
|---|---|
| Formato | `.glb` único; Y‑up; metros; origem nos pés; frente para +Z |
| Rig (Mixamo) | Bones humanoides; braços/antena/ombros preservados |
| Clipes | `Idle` (loop), `Thinking` (loop), `Processing` (loop), `Loading` (loop), `Success` (one‑shot), `Error` (one‑shot), `Wave` (one‑shot, opcional) — in‑place, sem root motion |
| Materiais | PBR; emissivo nos olhos/tela (ciano/azul `#0f4c81`/`#57a0d3`) |
| Compressão | Draco ou Meshopt + texturas ≤ 2048 (KTX2 opcional) |
| Tamanho | ≤ 3 MB (alvo ≤ 1,5 MB) |
| Entrega | `frontend/src/assets/models/jota/jota.glb` + `jota-poster.webp` (fallback) |

## Contrato do componente (frontend)

```ts
export type JotaEstado = 'idle' | 'carregando' | 'pensando' | 'processando' | 'sucesso' | 'erro';

<app-jota-3d [estado]="..." [tamanho]="'sm'|'md'|'lg'" [decorativo]="bool" [modeloUrl]="..." (falha)="..." />
```

Regras:

- `estado` é **input**; o componente só reflete — nunca decide o resultado.
- Mapa estado→clipe: idle→`Idle`, carregando→`Loading`, pensando→`Thinking`, processando→`Processing`, sucesso→`Success` (+ volta a `Idle`), erro→`Error` (+ volta a `Idle` após 2 s).
- Sem WebGL, GLB ausente/falho ou `prefers-reduced-motion` (modo 2D) → fallback `<img>` (`jota-poster.webp` ou `jota.png`) com `alt` por estado; `falha` emite para a página registrar.
- Loop de render em `NgZone.runOutsideAngular`; RAF pausado em `document.hidden` e fora da viewport (`IntersectionObserver`); dispose completo no destroy (`cancelAnimationFrame`, geometrias/materiais/texturas, `renderer.dispose()`, `forceContextLoss()`); um único contexto WebGL por componente; DPR ≤ 2.
- `import('three')` **dinâmico** dentro do componente (chunk lazy; zero no inicial).
- Canvas com `aria-hidden`; estado sempre comunicado em texto pela própria tela (já existe `role="status"`).

## Árvore de arquivos

| Caminho | Status | Responsabilidade |
|---|---|---|
| `docs/planos/jota-3d/00-PLANO-E-ESFORCO.md` | novo | Este plano (status feito/pendente) |
| `docs/planos/jota-3d/01-FASE-5-GERACAO-DO-MODELO.md` | novo | Roteiro de geração do GLB + checklist de validação |
| `docs/planos/jota-3d/referencias/pose-*.png` | novo | 6 recortes de pose (insumo image‑to‑3D e pose‑chave de cada clipe) |
| `frontend/package.json` | alterar (fase 3) | `three` (pinado) |
| `frontend/public/web.config` | alterar (fase 4) | `mimeMap` `.glb`/`.bin`/`.ktx2` |
| `frontend/src/assets/models/jota/jota.glb` | novo (externo, fase 5) | Modelo articulado |
| `frontend/src/assets/models/jota/jota-poster.webp` | novo (externo, fase 5) | Fallback otimizado |
| `frontend/src/app/shared/components/jota-3d/*` | novo (fase 1) | Componente, tipos, loader, specs |
| `frontend/src/app/features/chat/chat-page.*` | alterar (fase 2) | Integração `/chat` |
| `frontend/src/app/features/chat/jota-widget/*` | alterar (fase 6) | Widget com 3D |
| `docs/referencia/frontend.md`, `docs/arquitetura/componentes.md`, `docs/operacao/deploy.md`, `docs/operacao/troubleshooting.md` | alterar (fase 8) | Docs via docs-writer |

## Fases e esforço

| # | Fase | Entrega | Esforço | Depende de | Aceite |
|---|---|---|---|---|---|
| 1 | Contrato + componente sem `three` | `jota-3d.*` com estados, fallback e specs | 4 h | — | Karma verde; `validate.ps1` verde |
| 2 | Integração `/chat` | Herói + estado vazio reagem a `carregando`/`erro`/sucesso | 3 h | 1 | Estados corretos; sem regressão |
| 3 | `three` lazy + viewer + mixer + flag **off** | Chunk lazy isolado; flag `JOTA_3D` desligada | 10 h | 1 | Bundle inicial inalterado; chunk ≤ 200 KB gzip; dispose testado |
| 4 | MIME IIS + estrutura de assets | `.glb` servido no IIS HML | 2 h | — | `GET /assets/models/jota/jota.glb` → 200 |
| 5 | Modelo `jota.glb` (externo: IA→Mixamo→Blender) + validação/otimização | GLB dentro do contrato | 4 h (agente) + 2–5 d (externo) | — | Clipes/escala/tamanho validados |
| 6 | Ativar 3D no `/chat` e no widget | Flag on nos dois pontos | 6 h | 3+5 | FPS p95 ≤ 33 ms; 1 contexto; pausa em aba oculta |
| 7 | (Opcional) Ideias `analisandoIa` | 3º ponto | 2 h | 3+5 | — |
| 8 | Testes finais, perf/memória, docs, smoke | Fechamento | 4 h | todos | 20 ciclos sem vazamento; docs atualizados; smoke IIS |

**Totais:** código ≈ **33 h** (sem fase 7: ≈ 31 h); calendário ≈ **5–7 dias úteis**; modelo externo é o caminho crítico (paralelizável com as fases 1–4).

## Riscos e mitigação

| Risco | Mitigação |
|---|---|
| GLB atrasar | Fases 1–4 não dependem dele; flag mantém 3D off |
| GLB fora do contrato | Validação obrigatória (fase 5) antes de ligar flag |
| IIS 404 em `.glb` | `mimeMap` + smoke (fase 4) |
| Vazamento de GPU | Dispose + teste de 20 ciclos |
| Bundle inicial | Gate: fase 3 aborta se o inicial crescer |
| Change detection por frame | `runOutsideAngular` no RAF |

## Critérios de aceite globais

- Bundle inicial **sem aumento**; `three` só em chunk lazy.
- GLB ≤ 3 MB com os clipes nomeados.
- 1 contexto WebGL após 20 ciclos abrir/fechar; memória estável.
- `prefers-reduced-motion` → fallback 2D sem loop.
- Falha de WebGL/GLB → fallback com `alt` e texto de status intacto.
- `scripts/validate.ps1` verde em cada fase; smoke IIS servindo `.glb`.

## Status da execução — o que já foi feito e o que está pendente

### Feito (21/10/2026)

- [x] Plano aprovado e gravado (este arquivo)
- [x] **Fase 1** — componente `app-jota-3d` + specs (`shared/components/jota-3d/`, 10 specs; suíte Karma do repo em 49)
- [x] **Fase 2** — `/chat`: herói e estado vazio ligados aos estados reais (`carregando`→pensando, `erro`→erro, flash de sucesso 1,8 s)
- [x] **Fase 3** — `three@0.186.1` + `@types/three` (dev); loader lazy `jota-3d.loader.ts` (mixer, RAF fora da zone, pausa em aba oculta/offscreen, dispose completo); gate `JOTA_3D_HABILITADO = false` — **build confirma `WebGLRenderer` só em chunks lazy**
- [x] **Fase 4** — `mimeMap` `.glb`/`.gltf`/`.bin`/`.ktx2` no `frontend/public/web.config`
- [x] **Fase 5 (preparação)** — 6 referências recortadas e verificadas em `referencias/` + roteiro de geração em `01-FASE-5-GERACAO-DO-MODELO.md`
- [x] **Fase 6 (código)** — `<app-jota-3d>` no `/chat` e no cabeçalho do painel do widget (`tamanhoPx=34`); **ainda renderiza 2D** porque o gate está off
- [x] Docs canônicos atualizados (docs-writer): `referencia/frontend.md`, `arquitetura/componentes.md`, `operacao/deploy.md` (checklist pré-ativação), `operacao/troubleshooting.md`, `README.md`
- [x] Gate `scripts/validate.ps1` verde na última execução; Karma 49/49

### Pendente (para o futuro)

- [ ] **Fase 5 (GLB)** — gerar/exportar `frontend/src/assets/models/jota/jota.glb` pelo pipeline de `01-FASE-5-GERACAO-DO-MODELO.md` (imagem→3D→rig→clipes→export)
- [ ] **Fase 6 (ativação)** — validar o GLB + smoke do `.glb` no IIS; ligar `JOTA_3D_HABILITADO = true`
- [ ] **Fase 7 (opcional)** — levar o mascote à tela de análise de Ideias (`analisandoIa`)
- [ ] **Fase 8 (fechamento)** — perf/memória (20 ciclos sem vazamento), smoke final, docs do estado ativo
- [ ] **Commit** — o trabalho da feature está no working tree (não commitado); preservar com `feat(jota-3d)` quando retomar

### Observações da execução (fases 1–4, 21/10/2026)

- O `jota-widget` fica no `MainLayout` (eager), então o **componente** `app-jota-3d` (HTML/SCSS/TS, ~2,5 kB gzip) entra no bundle inicial; o **`three` não** (criterio de aceite da fase 3 cumprido).
- `three` não publica tipos nativos: `@types/three` foi adicionado como devDependency.
- Animação one-shot (`sucesso`/`erro`) usa o evento `finished` do **mixer** (o tipo `AnimationAction` do `@types/three` não expõe listeners).
- `tamanhoPx` (input numérico) resolve tamanhos fora do contrato `sm|md|lg` (ex.: avatar de 34 px no widget) via `--jota-size` inline.
- ⚠️ No working tree desta revisão `frontend/src/assets/images/jota.png` **não está presente** (existe só `assets/images/JOTA/JOTA-1.png`), embora seja o `posterUrl` default do componente, o `src` dos chips de persona e o do FAB do widget — confirmar/restaurar o asset antes do smoke visual.
