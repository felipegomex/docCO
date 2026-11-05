---
id: 17-12-atendimento-whatsapp.11
titulo: "WA-90 — UX da tela de números: filtros na URL, modal, menu de ações e exclusão em duas etapas"
tipo: referencia
area: qa
sistema: central
tags: [qa, whatsapp, atendimento, historias]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features/whatsapp
  - backend/Central_BackEnd/Controllers/WhatsApp
  - backend/Central_BackEnd/Services/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/WhatsAppAtendimentoTests.cs
  - frontend/src/app/features/whatsapp/pages/kanban/whatsapp-kanban.component.spec.ts
tokens_estimados: 1026
ficha: docs/modulos/whatsapp.md
---
> Parte de [12. Atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### WA-90 — UX da tela de números: filtros na URL, modal, menu de ações e exclusão em duas etapas

- **Objetivo:** operar a administração dos números com uma leitura única de conexão, recorte compartilhável, formulário em modal e exclusão que mostra o histórico antes de apagá-lo.
- **Pré-condições:** perfil **Administrador**; pelo menos dois números — um **com** histórico (conversas/atendimentos/mensagens) e um **sem** — e pelo menos um setor ativo na `tbfuncao`.
- **Passos:** 1. Abrir `/whatsapp/numeros` com a aba em rede lenta e conferir o esqueleto. 2. Filtrar por busca, setor e conexão e observar a URL (`busca`, `setor`, `conexao`). 3. Usar voltar/avançar do navegador e o botão **Limpar**. 4. Abrir **Novo número**, testar Esc/backdrop, o foco inicial, o **Salvar** desabilitado e o aviso de setor ativo. 5. Editar um número pelo menu **⋯**. 6. Abrir o menu e testar clique fora, rolagem, resize e Esc; conferir os itens. 7. Excluir o número **sem** histórico. 8. Excluir o número **com** histórico.
- **Resultado esperado:** `NumerosWhatsappComponent` mantém `carregando()` com três blocos `wa-esqueleto` (`role="status"`), filtros em `filtroBusca`/`filtroSetor`/`filtroConexao` sincronizados por `sincronizarFiltrosComUrl`/`atualizarUrlFiltros` (`replaceUrl` na busca; setor/conexão empilham uma entrada), contagem `numerosFiltrados().length` de `numeros().length` e estados vazios com **"Cadastrar o primeiro"** ou **"Limpar filtros"**. O formulário abre por `formAberto` em modal com `role="dialog"`, `aria-modal`, foco em `#descricao`, fechamento por Esc (`fecharCamadasComEsc`) ou backdrop, e `Salvar` desabilitado até `formValido()`; sem setores, o aviso linka `/sistema/funcoes-administrativas`. A linha traz só **Ler QR** e o menu **⋯**; o menu é `position: fixed` (não é cortado pelo `overflow` da tabela), fecha em clique fora/scroll/resize/Esc e oferece Iniciar sessão (desabilitado quando `estado === 'conectado'` ou há ação em curso), Parar sessão, Reconectar, Ler QR, Editar cadastro, Ativar/Desativar cadastro e Excluir número. Durante uma ação, a linha mostra "atualizando…" e `aria-busy`. A exclusão abre modal próprio: o primeiro `DELETE` vai sem `forcar`; **sem histórico** conclui na primeira confirmação; **com histórico** o `409 NUMERO_COM_HISTORICO` exibe a `mensagem` com as contagens e só então o botão vira **"Excluir com histórico"** (`forcar=true`). O resumo do topo (`resumo()`) mostra conectados, com problema, atendimentos ativos e setores; os cards aplicam o filtro correspondente, os chips filtram por setor e o alerta de "sem setor" abre o recorte `setor=-1`. A coluna **Conexão** mostra o badge de `conexaoPorNumero` e o `<details>` com Central, Gateway, Acessível, Último evento e Erro; `tempoRelativo` ("agora", "há 15 min", "há 3 h", "há 2 d") tem `title` com a data/hora absoluta, e o telefone formatado tem botão de copiar via Clipboard API (somente dígitos).
- **Estado atual/bloqueio:** cenário derivado do código, **sem QA de navegador** nesta revisão. O único teste automatizado ligado a esta tela é `whatsapp.utils.spec.ts`, que cobre `tempoRelativo` (menos de 1 min → "agora", minutos/horas/dias e entrada inválida → "-"); filtros, modal, menu e exclusão não têm spec. A mensagem do `409` vem do backend (`err.error.mensagem`); sem ela, a tela usa o texto padrão de impacto.
- **Rotas/endpoints/tabelas:** `/whatsapp/numeros`, `/sistema/funcoes-administrativas`; `GET /api/v1/whatsapp/numeros`, `GET /api/v1/whatsapp/saude`, `GET /api/v1/whatsapp/monitor`, `POST/PATCH /api/v1/whatsapp/numeros`, `DELETE /api/v1/whatsapp/numeros/{id}?forcar=`; `tbwhatsapp_numero`, `tbfuncao`.
- **Status/automação:** **cenário derivado do código**; `whatsapp.utils.spec.ts` cobre `tempoRelativo`; não há teste de frontend do componente.

