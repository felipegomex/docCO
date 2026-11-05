---
titulo: "Fichas de módulo"
status: "índice de fichas"
revisado_em: "2026-10-30"
---

# Fichas de módulo (`docs/modulos/`)

A ficha é o **mapa de implementação** do módulo: diz onde alterar, quais
contratos não podem quebrar, as armadilhas e os testes. Ela **aponta** para o
documento profundo (`docs/arquitetura/`, `docs/referencia/`, `docs/operacao/`) e
não o duplica. Entry point: [`docs/MAP.md`](../MAP.md).

## Regras (TEMPLATE)

Criar ficha nova copiando [TEMPLATE.md](./TEMPLATE.md): 14 seções obrigatórias,
`arquivo#símbolo` na seção 9, fonte por linha e fato não confirmado na seção 14.
Tamanho-alvo ≤ 2 mil tokens; `check-docs.ps1` valida arquivo e símbolo citados.

## Fichas (9)

| Módulo | Ficha | Status | Tokens |
|---|---|---|---|
| Atendimento WhatsApp | [whatsapp.md](./whatsapp.md) | ativo (2026-10-28) | ~2.0k |
| Implantação/Kanban | [kanban.md](./kanban.md) | ativo (2026-10-28) | ~2.0k |
| Auth, RBAC e Sistema | [auth-rbac-sistema.md](./auth-rbac-sistema.md) | ativo (2026-10-28) | ~2.0k |
| Acessos Empresas | [acessos.md](./acessos.md) | ativo (2026-10-28) | ~2.0k |
| Dashboard Integrado | [dashboard.md](./dashboard.md) | ativo (2026-10-28) | ~2.0k |
| Ideias, Melhorias e Erros | [ideias.md](./ideias.md) | ativo (2026-10-28) | ~2.0k |
| Chat e Conhecimento | [chat-conhecimento.md](./chat-conhecimento.md) | ativo (2026-10-30) | ~2.9k |
| Database Explorer | [database-explorer.md](./database-explorer.md) | ativo (2026-10-28) | ~2.0k |
| Agenda | [agenda.md](./agenda.md) | ativo (2026-10-28) | ~1.9k |

Não há mais módulo com `pendente-de-ficha` na Fase 2. As pendências das fichas
foram fechadas em 2026-10-29 (docs, policies, visibilidade e testes); o
faturamento do Dashboard segue bloqueado por decisão do Financeiro.

## Artefatos relacionados

- Roteador de contexto: [`docs/MAP.md`](../MAP.md) (gerado por `scripts/gen-docs-map.ps1`).
- Índice máquina-legível: [`docs/index.yaml`](../index.yaml).
- Glossário único: [`docs/glossario.md`](../glossario.md).
- ADRs: [`docs/decisoes/`](../decisoes/README.md).
