---
id: sec.07-3-2-auditoria
titulo: "3.2 Auditoria"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 595
ficha: docs/modulos/auth-rbac-sistema.md
---
> Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 3.2 Auditoria

### AU-01 — Consultar as trilhas de acessos e implantação

- **Objetivo:** Permitir que administrador ou gestor de função consulte as duas trilhas de auditoria — revelações de credencial e eventos `CREATE`/`UPDATE`/`DELETE` da implantação — com filtros e paginação.
- **Pré-condições:** operador autenticado; para ver dados, ser administrador ou gestor de função (`tbgestor`) — a decisão é do backend.
- **Passos:** 1. Abrir `/auditoria` pelo botão **Auditoria** no menu do usuário do header. 2. Na aba **Acessos**, filtrar por usuário e período, paginar e recarregar. 3. Trocar para a aba **Implantação**, filtrar por entidade, ação, usuário e período, paginar. 4. Expandir um evento para ler o antes/depois em JSON. 5. Repetir como operador sem papel (esperado: aviso "sem acesso").
- **Resultado esperado:** cada aba lista a trilha correspondente em ordem decrescente (página de 20 itens, com anterior/próxima); a aba Implantação mostra entidade, ação (badge), usuário e data, com detalhe expansível de antes/depois formatado (texto cru quando não é JSON válido). Filtros reaplicam a partir da página 1; limpar filtros recarrega. Sessão expirada avisa para entrar de novo; erro genérico usa toast. Operador autenticado sem papel recebe `403` do backend e vê o aviso "sem acesso" em vez de erro genérico — **não há guard de papel no frontend de propósito**, para os dois lados não divergirem.
- **Estado atual/bloqueio:** sem limitação específica identificada nesta inspeção; a cobertura é por papel no backend, não por rota no cliente.
- **Rotas/endpoints/tabelas:** `/auditoria`; `GET /api/v1/auditoria/acessos` (`usuario?`, `inicio?`, `fim?`, `page?`, `pageSize?`) e `GET /api/v1/auditoria/implantacao` (`entidade?`, `acao?`, `usuario?`, `inicio?`, `fim?`, `page?`, `pageSize?`), ambos `[Authorize]` com `403` para não-admin não-gestor; `tbauditoriaacesso` e `tbauditoriaimplantacao`.
- **Status/automação:** **cenário documentado**; não há teste automatizado da tela.

