---
title: "Histórias de Telas — QA"
description: "Cenários funcionais consolidados por tela, com rotas, endpoints, tabelas e nível de automação confirmados na árvore de trabalho."
date: "2026-10-27"
language: "pt-BR"
status: "cenário documentado"
---

# Histórias de Telas — QA

> Documento produzido por inspeção estática da árvore de trabalho, com revisão das telas do Dashboard Integrado em **2026-09-28** e atualizações incrementais desde então. Roteiros anteriores foram consolidados aqui e removidos em 09/10 (histórico no Git), sem tratar documentação arquivada como fonte superior ao código.
>
> **Regra de status:** `cenário documentado` significa que objetivo, passos e resultado foram derivados do código; não significa que o cenário foi executado. `validado em teste` significa que existe um artefato de teste automatizado que cobre o comportamento indicado; também não afirma que a suíte foi executada durante a criação deste documento.
>
> Não houve, para este documento, execução de navegador, build, suíte de testes, QA manual, deploy ou smoke. A classificação de automação vem do inventário de testes presente no repositório.
>
> **Estado atual/bloqueio:** cada história separa o comportamento desejado do estado observado no código. “Sem limitação específica” significa que nenhum bloqueio foi identificado nesta inspeção; “bloqueada/limitação conhecida” significa que o cenário não deve ser tratado como garantido até a correção indicada.
>
> **Consumo in-app:** `/faq-sistema` (rota canônica desde 2026-10-20; `/historias-telas` virou redirect para ela) renderiza o arquivo gerado `historias-telas.data.ts` (`scripts/extract-historias-telas.ts`, cuja fonte é o conjunto `docs/referencia/qa/*.md` (esta pasta, em ordem numérica). A `/historias-telas` exibe seção de contexto JCA/Actyon, busca (incluindo rotas/endpoints/tabelas), filtros por grupo e status com contagem, expandir/recolher todos, e badge de status colorido por tipo; a `/faq-sistema` (docs-api fase 4) organiza o mesmo dado em catálogo por tela do sistema.


## Partes

- [Critérios transversais confirmados](./01-criterios-transversais-confirmados.md)
- [Índice de telas para `Ctrl+F`](02-indice-de-telas-para-ctrl-f/INDEX.md)
- [1. Login/Base](./03-1-login-base.md)
- [2. Conhecimento/Ferramentas](04-2-conhecimento-ferramentas/INDEX.md)
- [3. Implantação](05-3-implantacao/INDEX.md)
- [3.1 Kanban Geral](./06-3-1-kanban-geral.md)
- [3.2 Auditoria](./07-3-2-auditoria.md)
- [3.3 Sistema — RBAC e operadores](./08-3-3-sistema-rbac-e-operadores.md)
- [4. Database](./09-4-database.md)
- [5. Empresa/Agenda](./10-5-empresa-agenda.md)
- [6. Admin/Kanban F](./11-6-admin-kanban-f.md)
- [7. Chat e Conhecimento](./12-7-chat-e-conhecimento.md)
- [8. Dashboard](./13-8-dashboard.md)
- [9. Acessos Empresas](./14-9-acessos-empresas.md)
- [10. Ideias, Melhorias e Erros](./15-10-ideias-melhorias-e-erros.md)
- [11. Início (antigo Planejamento)](./16-11-inicio-antigo-planejamento.md)
- [12. Atendimento WhatsApp](17-12-atendimento-whatsapp/INDEX.md)
- [13. Navegação por setor, visões gerais e Base de Conhecimento (2026-10-22)](./18-13-navegacao-por-setor-visoes-gerais-e-base-de-conhecimento-2026-10-22.md)
- [Regressão/histórico — funcionalidades desativadas ou sem rota ativa](./19-regressao-historico-funcionalidades-desativadas-ou-sem-rota-ativa.md)
- [Contagem final](./20-contagem-final.md)
- [Limitações e pontos que não puderam ser confirmados](./21-limitacoes-e-pontos-que-nao-puderam-ser-confirmados.md)
- [11. Apêndice — por que cada tabela existe](./22-11-apendice-por-que-cada-tabela-existe.md)
- [Arquivos de referência](./23-arquivos-de-referencia.md)
