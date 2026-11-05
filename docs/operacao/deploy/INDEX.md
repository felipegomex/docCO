---
title: "Deploy e publicação no IIS — Central de Operação"
description: "Runbook de validação, empacotamento, publicação, migrations, rollback e smoke do monorepo."
date: "2026-10-27"
language: "pt-BR"
status: "canônico"
---

# Deploy e Publicação no IIS — Central de Operação

Runbook consolidado do monorepo único, conferido contra o código, [`scripts/validate.ps1`](../../../scripts/validate.ps1), [`scripts/deploy/deploy.ps1`](../../../scripts/deploy/deploy.ps1) e [`scripts/smoke.ps1`](../../../scripts/smoke.ps1). Instruções antigas conflitantes foram consolidadas aqui e removidas em 09/10.

**Nenhum segredo é reproduzido aqui.** Forneça a credencial do servidor interativamente por `Get-Credential` ou por um mecanismo seguro aprovado; nunca envie a senha pela linha de comando, terminal compartilhada ou mensagem.

> **Credencial do servidor — comportamento atual do script:** `scripts/deploy/deploy.ps1` **não** contém credencial fixa nem fallback (a senha fixa anterior foi removida por exposição no histórico do Git). A publicação **exige** `-SenhaRemota` (`SecureString`) explícito e aborta sem ele. Execute sempre com `Get-Credential` ou `SecureString` de secret manager; nunca informe a senha como texto na linha de comando.


## Partes

- [Índice](./01-indice.md)
- [1. Escopo e regra de ouro](./02-1-escopo-e-regra-de-ouro.md)
- [2. Fonte de verdade e estrutura](./03-2-fonte-de-verdade-e-estrutura.md)
- [3. Pré-requisitos](./04-3-pre-requisitos.md)
- [4. Ambientes, portas e Chat/Conhecimento/reverse proxy](./05-4-ambientes-portas-e-chat-conhecimento-reverse-proxy.md)
- [5. Estrutura do IIS](./06-5-estrutura-do-iis.md)
- [6. Pipeline obrigatório](./07-6-pipeline-obrigatorio.md)
- [7. Migrations antes da publicação](08-7-migrations-antes-da-publicacao/INDEX.md)
- [8. Regras de package e publicação](./09-8-regras-de-package-e-publicacao.md)
- [9. `BUILD_INFO` e hash](./10-9-build-info-e-hash.md)
- [10. Preservação de configuração](./11-10-preservacao-de-configuracao.md)
- [11. Backup e rollback](./12-11-backup-e-rollback.md)
- [12. Branches, tags e versionamento](./13-12-branches-tags-e-versionamento.md)
- [13. Troubleshooting de deploy](./14-13-troubleshooting-de-deploy.md)
- [14. Limites conhecidos e fatos corrigidos](./15-14-limites-conhecidos-e-fatos-corrigidos.md)
- [15. Checklist de conclusão](./16-15-checklist-de-conclusao.md)
- [Arquivos relacionados](./17-arquivos-relacionados.md)
