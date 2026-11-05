# Fase 5 — Geração do modelo 3D do JOTA (referências prontas)

> **Status: referências preparadas (21/10/2026). Geração e exportação do `.glb` são o
> passo pendente — executados fora do repositório (usuário/artista), conforme pipeline aprovado.**

## 1. Insumos disponíveis (pasta `referencias/`)

Recortes da folha `frontend/src/assets/images/JOTA/JOTA-1.png` (4000×2250), separados por
componentes conexos com limpeza por rótulo (pose vizinha e artefatos — mancha superior e
bolinha azul — removidos para branco). Verificados visualmente, um a um.

| Arquivo | Clique do contrato | Pose (referência de animação) |
|---|---|---|
| `pose-idle.png` | `Idle` | braços cruzados, em pé |
| `pose-thinking.png` | `Thinking` | mão no queixo |
| `pose-processing.png` | `Processing` | correndo |
| `pose-loading.png` | `Loading` | sentado no laptop |
| `pose-success.png` | `Success` | acenando, feliz |
| `pose-error.png` | `Error` | cara triste + joinha |

Também disponíveis: a folha original (`JOTA-1.png`) e o poster usado no fallback 2D
(`frontend/src/assets/images/jota.png`).

## 2. Pipeline aprovado (opções)

**Opção A — serviços com rig/animação integrados (mais rápido):**
1. **Image‑to‑3D** (Meshy / Tripo3D / Rodin) usando **`pose-idle.png`** como base — é a pose
   de corpo inteiro mais neutra em pé. Prompt sugerido: *cute white and blue robot mascot,
   rounded soft shapes, black glossy face screen with glowing cyan eyes and smile, blue
   antenna, letter J logo on chest, chibi proportions, clean game‑ready character*.
2. **Auto‑rig** na própria ferramenta (Meshy Rig / Tripo) — as poses do JOTA têm braços
   colados/cruzados, então o rig automático de Mixamo exigiria reposer em A‑pose no Blender
   antes do upload.
3. **Clipes:** presets do serviço/Mixamo adaptados (`Idle`, `Running`) + clipes autorais no
   Blender referenciando as poses `loading`, `thinking`, `success`, `error`.
4. **Export `.glb`** (glTF binary), sem câmera/luz/plano de chão, Y‑up, origem nos pés, frente +Z.

**Opção B — artista em Blender:** modelagem fiel às 6 referências → rig humanoid → os 6 clipes
usando cada `pose-*.png` como pose‑chave → export.

## 3. Contrato de entrega (igual ao plano `00-PLANO-E-ESFORCO.md`)

- Arquivo: `frontend/src/assets/models/jota/jota.glb` (+ `jota-poster.webp` opcional).
- Clipes com **nomes exatos**: `Idle`, `Thinking`, `Processing`, `Loading`, `Success`, `Error`
  (o loader faz lookup por nome, case‑insensitive; sem `Idle` o fallback é não animar).
- One‑shot: `Success` e `Error` (o loader volta a `Idle` pelo evento `finished` do mixer).
- Tamanho ≤ 3 MB (alvo ≤ 1,5 MB; Draco/Meshopt + texturas ≤ 2048).
- Escala ~1 m; enquadramento é defensivo (o loader ajusta a câmera ao bounding box).

## 4. Checklist de validação pós‑entrega (antes de ligar o gate)

- [ ] `gltf-validator` sem erros; clipes listados com os nomes do contrato.
- [ ] Tamanho do arquivo dentro do limite.
- [ ] Preview (gltf‑viewer) mostra materiais/emissivo corretos, sem chão/câmera embutidos.
- [ ] `GET /assets/models/jota/jota.glb` → **200** no IIS HML (MIME `.glb` já no `web.config`).
- [ ] Ligar `JOTA_3D_HABILITADO = true` (`jota-3d.types.ts`) e validar `/chat` + widget.
- [ ] Fase 8: 20 ciclos sem vazamento de contexto, pausa em aba oculta, `validate.ps1` verde.

## 5. Estado das demais fases

Ver o checklist de `00-PLANO-E-ESFORCO.md` — fases 1–4 e o código da fase 6 estão concluídos
(gate desligado, sistema comporta‑se como 2D); pendem somente este `.glb`, a ativação e o
fechamento (fase 8).
