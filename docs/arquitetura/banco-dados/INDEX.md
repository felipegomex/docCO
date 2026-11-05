---
title: "Estrutura de dados"
description: "Entidades, tabelas, relacionamentos, migrations e integrações de dados do monorepo atual."
date: "2026-10-30"
language: "pt-BR"
status: "canônico"
revisado_em: "2026-10-30"
---

# Estrutura de dados

Inventário calculado a partir de [AppDbContext.cs](../../../backend/Central_BackEnd/Data/AppDbContext.cs), dos modelos e do [AppDbContextModelSnapshot.cs](../../../backend/Central_BackEnd/Migrations/AppDbContextModelSnapshot.cs). Não reproduz DDL, strings de conexão, senhas, chaves ou qualquer valor de configuração.


## Partes

- [Índice para busca](./01-indice-para-busca.md)
- [Padrão de nomenclatura (canônico)](./02-padrao-de-nomenclatura-canonico.md)
- [Contagens confirmadas](./03-contagens-confirmadas.md)
- [Fontes de verdade](./04-fontes-de-verdade.md)
- [Provider de persistência](./05-provider-de-persistencia.md)
- [Inventário de entidades e tabelas](06-inventario-de-entidades-e-tabelas/INDEX.md)
- [Relacionamentos confirmados](./07-relacionamentos-confirmados.md)
- [Coluna gerida: `tblead.LEAD_ContatoVisualizado`](./08-coluna-gerida-tblead-lead-contatovisualizado.md)
- [Colunas geridas: funil comercial com produtos (2026-10-08)](./09-colunas-geridas-funil-comercial-com-produtos-2026-10-08.md)
- [Migrations e tabelas históricas](10-migrations-e-tabelas-historicas/INDEX.md)
- [Fusão de `CC_Funcao` na legada `tbfuncao`](./11-fusao-de-cc-funcao-na-legada-tbfuncao.md)
- [Tabelas removidas e correção factual](./12-tabelas-removidas-e-correcao-factual.md)
- [Migrations carimbadas manualmente e armadilhas de histórico](./13-migrations-carimbadas-manualmente-e-armadilhas-de-historico.md)
- [Refresh tokens e auditoria](./14-refresh-tokens-e-auditoria.md)
- [Database Explorer](./15-database-explorer.md)
- [Segurança e dados sensíveis](./16-seguranca-e-dados-sensiveis.md)
- [Contagens e fatos confirmados no baseline](./17-contagens-e-fatos-confirmados-no-baseline.md)

## Inventário incremental — `tbconfiguracao_sistema` (2026-10-30)

Tabela de configuração com segredos do módulo Chat/Conhecimento. Entidade
`ConfiguracaoSistema` (mapeada em [AppDbContext.cs](../../../backend/Central_BackEnd/Data/AppDbContext.cs),
`DbSet` `ConfiguracoesSistema`), criada pela migration
[20261030170000_ConfiguracaoSistema.cs](../../../backend/Central_BackEnd/Migrations/20261030170000_ConfiguracaoSistema.cs)
e pelo SQL idempotente
[ChatConfiguracaoSistema_Idempotente.sql](../../../backend/Central_BackEnd/Migrations/Sql/ChatConfiguracaoSistema_Idempotente.sql),
que registra a migration em `__EFMigrationsHistory` quando a tabela já existe.

| Coluna | Tipo | Observação |
|---|---|---|
| `CFG_Chave` | `nvarchar(100)` | PK; nome lógico da configuração |
| `CFG_Valor` | `nvarchar(max)` | valor; quando `CFG_Segredo = 1`, guarda o conteúdo protegido (envelope) |
| `CFG_Segredo` | `bit` | marca o valor como segredo |
| `CFG_AtualizadoEm` | `datetime2` | última gravação |
| `CFG_AtualizadoPor` | `nvarchar(100)` | operador responsável ou `sistema` |

Chaves conhecidas (`ConfiguracaoSistemaChaves`): `acessos.chave_cifra`
(chave-mestra AES-256, cifrada pelo DataProtection) e `gitea.service_token`
(token de serviço do Gitea cifrado com AES-GCM, envelope `v1:iv:tag:ciphertext`).
Este documento não reproduz DDL nem valores de configuração. **Estado:** SQL
aplicado em HML/PROD em 2026-10-30 (4 migrations de chat); sem deploy.
