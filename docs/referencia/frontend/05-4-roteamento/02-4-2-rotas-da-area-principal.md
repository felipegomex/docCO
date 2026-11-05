---
id: 05-4-roteamento.02
titulo: "4.2 Rotas da área principal"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 1145
ficha: docs/modulos/README.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### 4.2 Rotas da área principal

Todas as linhas abaixo herdam `authGuard` do shell em `/`.

| URL | Componente ativo |
|---|---|
| `/` | `HomeComponent` |
| `/ferramentas` | `FerramentasComponent` |
| `/ferramentas/detalhe/:id` | `FerramentaDetalheComponent` |
| `/ferramentas/acessos` | redirect para `/acessos/clientes` (`pathMatch: full`) |
| `/ferramentas/faq` | **redirect para `/base-conhecimento?aba=glossario`** (`pathMatch: full`) — o Glossário virou aba da Base de Conhecimento |
| `/base-conhecimento` | `BaseConhecimentoComponent` — shell com abas **Stack**, **Glossário** e **Conhecimento Rápido** (sub-abas Resolver, SQL, Rede e Infra); `?aba=` (`stack` padrão, `glossario`, `conhecimento`) e `?trilha=` (`resolver`, `sql`, `rede`, `infra`) |
| `/stack` | **redirect para `/base-conhecimento?aba=stack`** (`pathMatch: full`) |
| `/agenda` | `AgendaComponent` |
| `/chat` | `ChatIframeComponent` (iframe do AnythingLLM autenticado por SSO) |
| `/conhecimento` | `ConhecimentoShellComponent` + lista de documentos; filhos `nova`, `configuracao`, `editor`, `diff` e `historico`, protegidos por `revisorGuardLazy` que consulta `GET /api/conhecimento/permissao`; `/conhecimento/configuracao` apresenta os mapeamentos de workspace e revisores para administração. **Rota sem item de menu** desde 2026-10-21 (ver §4.10) |
| `/cursos` | `CursosComponent` |
| `/cursos/detalhe/:id` | `CursoDetalheComponent` |
| `/trilhas/resolver` | **redirect para `/base-conhecimento?aba=conhecimento&trilha=resolver`** |
| `/trilhas/sql` | **redirect para `/base-conhecimento?aba=conhecimento&trilha=sql`** |
| `/trilhas/rede` | **redirect para `/base-conhecimento?aba=conhecimento&trilha=rede`** |
| `/trilhas/infra` | **redirect para `/base-conhecimento?aba=conhecimento&trilha=infra`** |
| `/visao-adm` | `VisaoAdmComponent` |
| `/visao-adm/detalhe/:id` | `VisaoAdmDetalheComponent` |
| `/fraseologia` | `FraseologiaComponent` |
| `/modelo-chamados` | `ModeloChamadosComponent` |
| `/politica` | `PoliticaComponent` |
| `/historias-telas` | **redirect para `/faq-sistema`** (`pathMatch: full`) — compatibilidade com o link antigo; o componente foi removido em 2026-10-20 |
| `/faq-sistema` | `FaqSistemaComponent` — **rota canônica** do catálogo de telas em cards (menu "Guia do Sistema" na seção **Sistema**), protegida por `permissaoGuard('aba.sistema.guia')` |
| `/empresa/onboarding` | `OnboardingHomeComponent` |
| `/empresa/onboarding/capitulo/:id` | `OnboardingCapituloComponent` |
| `/auditoria` | `AuditoriaComponent` (lazy por `auditoria.routes.ts`; duas abas somente leitura — acessos e implantação). Entrada pelo botão **Auditoria** no menu do usuário do header (visível a todo autenticado, sem filtro de papel) e breadcrumb `Auditoria` |
| `/gestores-ideias` | `GestoresComponent` (lazy por `gestores.routes.ts`, em `features/gestores/pages/gestores/`). **Não** é rota do shell `/admin` — ver §4.3 |
| `/sistema` | redirect para `/sistema/funcoes-administrativas` (`pathMatch: full`) |
| `/sistema/funcoes-administrativas` | `FuncoesAdministrativasComponent` (lazy por `sistema.routes.ts`) — cards com atalhos para Operadores, Gestores, **Permissões** (condicionado ao recurso `aba.sistema.permissoes`), Cadastros, Colunas do Kanban e Auditoria; `permissaoGuard('aba.sistema.funcoes-administrativas')` |
| `/sistema/operadores` | `OperadoresComponent` (lazy por `sistema.routes.ts`) — cadastro de operadores, cargo e setor; `permissaoGuard('aba.sistema.operadores')` |
| `/sistema/permissoes` | `PermissoesComponent` (lazy por `sistema.routes.ts`, `OnPush`) — **tela de controle da matriz de permissões**; `permissaoGuard('aba.sistema.permissoes')`. A lista de cargos vem de `GET /permissoes/cargos-gerenciaveis` (Master ou Tech Lead do setor/nível), e não mais de `GET /cargos`. Recursos novos de `Permissoes_Matriz_Idempotente.sql` aparecem automaticamente após o SQL ser aplicado |
| `/sem-acesso` | `SemAcessoComponent` — destino do `permissaoGuard` quando o recurso não está liberado; recebe `?recurso=` e explica qual permissão falta, em vez de voltar para `/` em silêncio |

`implantacao`, `database`, `dashboard`, `acessos`, `ideias` e `gestores-ideias` usam `loadChildren`; as demais telas são carregadas com `loadComponent`.

