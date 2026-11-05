---
id: 06-5-features-e-status-de-uso.indice
titulo: "5. Features e status de uso"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 300
ficha: docs/modulos/README.md
---

# 5. Features e status de uso

> Índice do documento dividido. Documento original: [06-5-features-e-status-de-uso.md](../06-5-features-e-status-de-uso.md).
> Ficha: [Fichas de módulo](../../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../../frontend.md).


| Diretório em `features` | Status confirmado |
|---|---|
| `acessos` | Ativa por `loadChildren` em `/acessos` → `/acessos/clientes`; **substituiu** a antiga `AcessosComponent` de 782 linhas em `/ferramentas/acessos` |
| `admin` | Ativa por `/admin` → `/admin/cadastros/tipos-projeto`. O componente e o serviço de dashboard administrativo foram **removidos** em 2026-10-20 (não tinham rota); o endpoint do backend continua registrado |
| `agenda` | Ativa por `/agenda` |
| `auditoria` | Ativa por `loadChildren` em `/auditoria` → `AuditoriaComponent` (duas abas: revelações de credencial e eventos da implantação, com filtros e paginação; **sem guard de papel** — sem permissão o backend devolve `403` e a tela avisa "sem acesso") |
| `auth` | Ativa por `/login` |
| `base-conhecimento` | Ativa por `/base-conhecimento` — shell com abas Stack, Glossário e Conhecimento Rápido; reaproveita `StackComponent`, `FaqComponent` e os componentes de trilha |
| `chat` | Ativa por `/chat` (`ChatIframeComponent`); iframe do AnythingLLM com SSO; sem widget global; sem item de menu "Agentes" na versão atual |
| `conhecimento` | Ativa: shell e lista, nova revisão, editor, diff e histórico. `revisorGuardLazy` respeita a resposta do backend em `/api/conhecimento/permissao`; falha ou falta de permissão redireciona para `/`. O editor também usa `unsavedChangesGuard`. **Sem item de menu desde 2026-10-21** (o Gitea não está configurado no HML) — a rota e o código continuam no repositório |
| `cursos` | Ativa por `/cursos` e detalhe |
| `dashboard` | Ativa por `loadChildren` em `/dashboard`; home, comercial, administrativo e financeiro |
| `ideias` | Ativa por `loadChildren` em `/ideias` → lista, nova, nova/:tipo, :id; mais `/admin/ideias/gestores`. **Em construção** — F8/F9/F10/F11 entregues, F6/F7/F12/F13/F14 pendentes, ainda não publicada |
| `database` | Ativa por `loadChildren` em `/database` |
| `empresa` | Ativa por `/empresa/onboarding` e capítulos |
| `faq` | Ativa como aba **Glossário** de `/base-conhecimento`; `/ferramentas/faq` redireciona para `/base-conhecimento?aba=glossario`. É o *Glossário Interno Actyon*, com 7 capítulos |
| `faq-sistema` | Ativa por `/faq-sistema` (plano docs-api, fase 4); standalone, `OnPush`, catálogo em cards com busca/filtros; consome o dado local `historiasTelas` (v1) — nenhum service próprio. **Rota canônica** desde 2026-10-20 (Fase 0): `/historias-telas` redireciona para cá e o item de menu é "Guia do Sistema". Desde a versão exibida no card: badge de versão (SemVer `v{{ versao }}`, commit e data local via `BuildInfoService`/`GET /version.json`, fallback `APP_VERSION`) e **mini sistema** de navegação por módulo (`abrirNoGrupo`, filtra por módulo e abre/rola o card) antes dos filtros; card ganhou `[id]={'fqs-card-' + h.id}` |
| `ferramentas` | Ativa por lista, detalhe e workspace; o card com `embutivel: true` navega para `/ferramentas/workspace?painel=:id` (iframe integrado ao layout, com comparação `?comparar=:id` e tela cheia interna) e o com `embutivel: false` abre nova aba |
| `fraseologia` | Ativa por `/fraseologia` |
| `home` | Ativa na rota `/` |
| `historias-telas` | **Só o arquivo de dados** `historias-telas.data.ts`, gerado por `scripts/extract-historias-telas.ts` a partir de `docs/referencia/qa-historias.md` — **não editar à mão**. A rota `/historias-telas` virou redirect para `/faq-sistema` e o componente foi removido em 2026-10-20 (Fase 0) |
| `implantacao` | Ativa por `loadChildren` em `/implantacao`; as telas do Kanban Geral (`/kanban`, `/kanban/gargalos`, `/kanban/configurar`, `/kanban/admin/colunas`) são rotas de topo em `features.routes.ts` e reaproveitam `pages/kanban`. O `DashboardComponent` também serve `/suporte/visao-geral`, `/desenvolvimento/visao-geral` e `/ciaa/visao-geral`, e `ProjetosComponent`/`ProjetoFormComponent`/`ProjetoDetalheComponent` servem `/ciaa/projetos*`, com `data.funcaoId` e `data.basePath` |
| `modelo-chamados` | Ativa por `/modelo-chamados` |
| `politica` | Ativa por `/politica` |
| `services` | Ativa como camada de serviços de acervos locais |
| `sistema` | Ativa por `loadChildren` em `/sistema` → `funcoes-administrativas` (cards de atalho), `operadores` (cadastro com cargo/setor, sem `DELETE`) e `permissoes` (tela de controle da matriz); todas com `permissaoGuard` por recurso |
| `stack` | Ativa como aba **Stack** de `/base-conhecimento`; `/stack` redireciona para `/base-conhecimento?aba=stack` |
| `trilha-infra` | Ativa na sub-aba **Infra** do Conhecimento Rápido; `/trilhas/infra` redireciona para `/base-conhecimento?aba=conhecimento&trilha=infra` |
| `trilha-rede` | Ativa na sub-aba **Rede**; `/trilhas/rede` redireciona para `/base-conhecimento?aba=conhecimento&trilha=rede` |
| `trilha-sql` | Ativa na sub-aba **SQL**; `/trilhas/sql` redireciona para `/base-conhecimento?aba=conhecimento&trilha=sql` |
| `trilhas` | Ativa na sub-aba **Resolver**; `/trilhas/resolver` redireciona para `/base-conhecimento?aba=conhecimento&trilha=resolver` |
| `visao-adm` | Ativa por lista e detalhe de procedimentos |
| `whatsapp` | Ativa por `loadChildren` em `/whatsapp` → Kanban da fila, **Minhas Conversas**, painel do atendimento com **painel do cliente**, **mídia sob demanda**, **Assistente IA** (dados reais: resumo, classificação, sugestões via `GET /ia/atendimentos/{id}/assistente-ia`), números (administrador), métricas (abas **Geral**, **Qualidade**, **Clientes**), respostas rápidas, **auditoria** (admin), **configuração IA** (admin), configuração, SLA, retenção e o **hub administrativo** com a **gestão de contatos** e o **catálogo de etiquetas** (Fase 08). **Implementado no código**; depende do gateway OpenWA e do backend habilitado |

## Partes

- [5.1 Módulo Acessos Empresas — inventário](./01-5-1-modulo-acessos-empresas-inventario.md)
- [Comportamentos que não devem ser "simplificados"](./02-comportamentos-que-nao-devem-ser-simplificados.md)
- [Navegação registrada para o módulo](./03-navegacao-registrada-para-o-modulo.md)
- [5.2 Módulo de atendimento WhatsApp — inventário](./04-5-2-modulo-de-atendimento-whatsapp-inventario.md)
- [5.2 Módulo de atendimento WhatsApp — inventário — introdução — parte 2](./05-5-2-modulo-de-atendimento-whatsapp-inventario-introducao-par.md)
- [Decisões que a implementação tomou (e que o código explica)](./06-decisoes-que-a-implementacao-tomou-e-que-o-codigo-explica.md)
- [Navegação registrada para o módulo](./07-navegacao-registrada-para-o-modulo.md)
