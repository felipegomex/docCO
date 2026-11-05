---
id: visao-geral.02
titulo: "1. Propósito"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 340
ficha: docs/modulos/README.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 1. Propósito

A Central de Operação é um portal interno que reúne, em uma única aplicação:

- conhecimento e conteúdo de atendimento;
- ferramentas e utilitários;
- gestão de credenciais de acesso das empresas (grupo → unidade → RDP, VPN, AnyDesk, banco e Actyon Web);
- agenda operacional;
- gestão de projetos, implantações, tarefas e o **Kanban Geral** (quadro por função de tarefas e projetos);
- **atendimento de WhatsApp multiatendente** (fila em Kanban, conversa, atribuição, métricas, administração de números e **atualização em tempo real** pelo hub SignalR);
- gestão de acesso por **cargo** (RBAC): permissões por recurso/ação, escopo Próprio/Função/Tudo e administração de operadores;
- consulta de estrutura de banco de dados;
- conteúdos institucionais da JCA;
- acesso ao assistente JOTA pela página de Chat e pelo widget global.

O produto combina conteúdo estático, dados cadastrados e operações em tempo de execução. A documentação deve distinguir essas três origens para não apresentar conteúdo local como API real.

