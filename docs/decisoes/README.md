---
titulo: "Registro de decisões (ADRs)"
status: "canônico"
data: "2026-10-18"
---

# Registro de decisões (ADRs)

Registro das decisões de arquitetura já tomadas, cada uma com contexto, decisão, consequências e fonte. Só viram ADRs decisões **explicitamente documentadas** nos canônicos; decisão sem registro verificável não vira ADR.

## Regras

1. **Numeração sequencial**: `NNN-titulo-em-kebab-case.md` (`001`, `002`, `003`, …). Nunca reusar número descartado.
2. **Imutabilidade**: um ADR **aceito não é editado**. Mudança na decisão → novo ADR que referencia o anterior. Erro factual pode ser corrigido por novo ADR, não por edição silenciosa do antigo.
3. Todo ADR tem as seções Contexto, Decisão, Consequências e Fonte; todo fato citado aponta a fonte (canônico ou `arquivo:símbolo`).

## Índice

| ADR | Título | Estado |
|---|---|---|
| [001-setor-para-funcao](./001-setor-para-funcao.md) | Eixo do Kanban Geral: Setor → Função | aceito |
| [002-criptografia-desligada](./002-criptografia-desligada.md) | Criptografia do Acessos Empresas desligada (`CifradorEmBranco`) | substituído por 005 |
| [003-sem-staging](./003-sem-staging.md) | Produção sem staging | aceito |
| [004-rbac](./004-rbac.md) | RBAC: permissão por recurso e por função, com herança de grupo e exceção individual | aceito |
| [005-cifra-aes-ativa](./005-cifra-aes-ativa.md) | Cifra AES-256-GCM ativa no Acessos Empresas | aceito |
