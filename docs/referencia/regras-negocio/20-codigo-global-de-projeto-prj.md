---
id: regras-negocio.20
titulo: "Código global de projeto `PRJ`"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 289
ficha: docs/modulos/kanban.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Kanban Geral](../../modulos/kanban.md).

## Código global de projeto `PRJ`

**Situação: parcial; é global, mas não é uma sequência segura para concorrência.**

- `ProximoCodigoAsync` lê todos os códigos `tbprojeto` que começam com `PRJ-`, extrai o sufixo numérico, encontra o maior e devolve `PRJ-` com quatro dígitos.
- O código é gerado no serviço de criação, sem escopo por cliente, tipo, equipe ou usuário. A rota `GET /api/v1/implantacao/projetos/proximo-codigo` apenas mostra a prévia.
- `PRJ_Codigo` possui índice único no modelo. Ainda assim, duas criações simultâneas podem calcular o mesmo sufixo; o índice pode rejeitar uma delas em vez de garantir nova geração.
- O campo `TipoProjeto.ClienteObrigatorio` existe, mas `ProjetoService.CriarAsync` não verifica esse sinalizador quando `ClienteId` é ausente. Não documentar “cliente obrigatório” como regra aplicada.

