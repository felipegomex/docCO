# Referencias visuais do JOTA (fase 5)

Recortes da folha `frontend/src/assets/images/JOTA/JOTA-1.png` (4000x2250),
separados por componentes conexos (scipy.ndimage.label) com limpeza por rotulo:
dentro de cada recorte, tudo que nao e a pose alvo — poses vizinhas que invadem
o bounding box e os 2 artefatos da folha (mancha superior, bolinha azul) — foi
pintado de branco. Os 6 arquivos foram verificados visualmente. Uso: insumo do
image-to-3D e pose-chave de cada clipe (docs/planos/jota-3d/01-FASE-5-GERACAO-DO-MODELO.md).

| Arquivo | Clique | Pose |
|---|---|---|
| pose-error.png | `Error` | cara triste + joinha |
| pose-idle.png | `Idle` | bracos cruzados |
| pose-thinking.png | `Thinking` | mao no queixo |
| pose-loading.png | `Loading` | sentado no laptop |
| pose-processing.png | `Processing` | correndo |
| pose-success.png | `Success` | acenando |
