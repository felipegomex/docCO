---
id: 06-5-features-e-status-de-uso.07
titulo: "Navegação registrada para o módulo"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, componentes]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 434
ficha: docs/modulos/README.md
---
> Parte de [5. Features e status de uso](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

#### Navegação registrada para o módulo

O módulo aparece em **4 arquivos** — esquecer um deixa a tela sem entrada:

| Arquivo | O que foi acrescentado |
|---|---|
| `features.routes.ts` | `path: "whatsapp"` com `loadChildren` para `@features/whatsapp/whatsapp.routes` |
| `shared/config/header-nav.config.ts` | Grupo **WhatsApp** na seção **SUPORTE**: "Atendimento" (`/whatsapp`, recurso `aba.suporte.whatsapp`), **"Minhas Conversas"** (`/whatsapp/minhas-conversas`, `aba.suporte.whatsapp`), "Indicadores" (`/whatsapp/metricas`, `aba.suporte.whatsapp.metricas`), **"Admin"** (`/whatsapp/admin`, `roles: ['Administrador']` — Fase 08), **"Auditoria"** (`/whatsapp/auditoria`, `roles: ['Administrador']`), **"Configuração IA"** (`/whatsapp/ia-configuracao`, `roles: ['Administrador']`), "Números" (`/whatsapp/numeros`, `aba.suporte.whatsapp.numeros`) e "Retenção de mídia" (`/whatsapp/retencao`, `roles: ['Administrador']` — fica fora do RBAC de propósito, porque o recurso ainda não existe no banco) |
| `core/services/busca-index.service.ts` | 2 entradas em `PAGINAS`: "Atendimento WhatsApp" (`/whatsapp`) e "Indicadores do WhatsApp" (`/whatsapp/metricas`). A tela de números **não** está na busca global — é infraestrutura de administrador |
| `shared/config/header-nav.config.ts` (dicas) | Cada item traz a `dica` exibida no item de menu |

