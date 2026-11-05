---
id: 02-indice-de-telas-para-ctrl-f.03
titulo: "Redirecionamentos ativos"
tipo: referencia
area: qa
sistema: central
tags: [qa, telas, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - docs/referencia/qa
testes:
  - docs/
tokens_estimados: 509
ficha: docs/modulos/README.md
---
> Parte de [Índice de telas para `Ctrl+F`](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### Redirecionamentos ativos

Não são telas independentes; aparecem no índice para evitar que um link direto antigo seja tratado como funcionalidade nova.

| Origem | Destino | Observação |
|---|---|---|
| `/implantacao` | `/implantacao/dashboard` | redirecionamento do módulo |
| `/implantacao/relatorio` | `/implantacao/dashboard` | relatório foi absorvido pelo dashboard |
| `/implantacao/kanban` | `/kanban` | o quadro passou a ser o Kanban Geral; o caminho antigo é atalho |
| `/implantacao/agenda` | `/agenda` | redirecionamento de compatibilidade |
| `/database` | `/database/visao-geral` | redirecionamento do shell |
| `/database/query-builder` | `/database/consultas` | nome legado redireciona ao Criador |
| `/empresa` | `/empresa/onboarding` | não existe tela própria de Empresa |
| `/ferramentas/acessos` | `/acessos/clientes` | endereço da tela antiga, que lia do Google Sheets |
| `/ferramentas/contra-senha` | `/ferramentas` | apelido legado |
| `/ferramentas/modelos` | `/modelo-chamados` | apelido legado |
| `/ferramentas/faq` | `/base-conhecimento?aba=glossario` | Glossário virou aba |
| `/stack` | `/base-conhecimento?aba=stack` | Stack virou aba |
| `/trilhas/*` | `/base-conhecimento?aba=conhecimento&trilha=*` | trilhas viraram sub-abas |
| `/suporte` | `/suporte/visao-geral` | visão do setor |
| `/desenvolvimento` | `/desenvolvimento/visao-geral` | visão do setor |
| `/ciaa` | `/ciaa/visao-geral` | visão do setor |
| `/whatsapp-flow` | `/fraseologia` | apelido legado |
| `/admin` | `/admin/cadastros` | depois redireciona para Tipos de Projeto |
| `/admin/cadastros` | `/admin/cadastros/tipos-projeto` | destino padrão |
| `/ideias` | `/ideias/lista` | redirecionamento do módulo |

