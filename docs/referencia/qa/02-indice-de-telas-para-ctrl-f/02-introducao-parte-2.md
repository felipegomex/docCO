---
id: 02-indice-de-telas-para-ctrl-f.02
titulo: "Introdução — parte 2"
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
tokens_estimados: 1960
ficha: docs/modulos/README.md
---
> Parte de [Índice de telas para `Ctrl+F`](./INDEX.md). Ficha: [Fichas de módulo](../../../modulos/README.md).

### Introdução — parte 2

| Grupo | Tela ou fluxo | Rota ativa | História | Status |
|---|---|---|---|---|
| Atendimento WhatsApp | Aplicar e remover etiquetas do atendimento | `/whatsapp/atendimento/:id` (aba Resumo) | WA-44 | cenário documentado (specs de etiqueta) |
| Atendimento WhatsApp | Conjunto igual de etiquetas não gera evento | `/whatsapp/atendimento/:id` (aba Resumo) | WA-45 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Alternar investigação sem mudar status/responsável/SLA | `/whatsapp/atendimento/:id` (aba Resumo) | WA-46 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Observação restrita ao atendimento atual | `/whatsapp/atendimento/:id` (aba Resumo) | WA-47 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Auditoria da observação sem o texto | `/whatsapp/atendimento/:id`, `/whatsapp/auditoria` | WA-48 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Etiqueta inativa continua visível no atendimento | `/whatsapp/atendimento/:id` (aba Resumo) | WA-49 | cenário documentado (teste/spec) |
| Atendimento WhatsApp | Operador recebe 403 ao gerenciar o catálogo | `/whatsapp/etiquetas` (API) | WA-50 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Card do Kanban Service Desk com contexto completo | `/whatsapp` | WA-51 | cenário documentado (teste/spec) |
| Atendimento WhatsApp | Cliente pendente no card e no filtro de empresa | `/whatsapp` | WA-52 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Etiquetas no card: até 4 + contador | `/whatsapp` | WA-53 | cenário documentado (spec) |
| Atendimento WhatsApp | Badge de investigação não muda a coluna | `/whatsapp` | WA-54 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Filtro server-side atualiza total e paginação | `/whatsapp` | WA-55 | cenário documentado (teste backend) |
| Atendimento WhatsApp | SLA crítico destacado e filtrado por dimensão | `/whatsapp` | WA-56 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Ações do card por status e permissão | `/whatsapp` | WA-57 | cenário documentado (spec) |
| Atendimento WhatsApp | Hub administrativo reúne indicadores e atalhos | `/whatsapp/admin` | WA-58 | cenário documentado (spec) |
| Atendimento WhatsApp | Filtros e paginação de contatos no servidor | `/whatsapp/admin/contatos` | WA-59 | cenário documentado (teste backend/spec) |
| Atendimento WhatsApp | Editar contato no painel e tratar conflito 409 | `/whatsapp/admin/contatos` | WA-60 | cenário documentado (teste backend/spec) |
| Atendimento WhatsApp | Histórico do contato em painel com link para o atendimento | `/whatsapp/admin/contatos` | WA-61 | cenário documentado (teste backend/spec) |
| Atendimento WhatsApp | Contato `@lid` marcado como identificador de privacidade | `/whatsapp/admin/contatos` | WA-62 | cenário documentado (teste backend/spec) |
| Atendimento WhatsApp | Atualização em tempo real sem esperar o polling | `/whatsapp`, `/whatsapp/atendimento/:id` | WA-63 | cenário documentado (teste backend/spec) |
| Atendimento WhatsApp | Reconexão do hub e recarga silenciosa | `/whatsapp` | WA-64 | cenário documentado (spec) |
| Atendimento WhatsApp | Indicador interno de digitação aparece e expira em 5 s | `/whatsapp/atendimento/:id` | WA-65 | cenário documentado (spec) |
| Atendimento WhatsApp | Nota interna não gera typing externo; mensagem gera | `/whatsapp/atendimento/:id` | WA-66 | cenário documentado (teste backend) |
| Atendimento WhatsApp | Polling continua funcionando com o SignalR desligado | `/whatsapp` | WA-67 | cenário documentado (spec) |
| Atendimento WhatsApp | Skeleton e erro com retry em blocos independentes | `/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/admin/contatos` | WA-68 | cenário a executar em HML (não é aceite) |
| Atendimento WhatsApp | Layout em 1440/1024/390 px com uma área por vez no mobile | `/whatsapp/atendimento/:id` | WA-69 | cenário a executar em HML (não é aceite) |
| Atendimento WhatsApp | Navegação inferior só no mobile | `/whatsapp/atendimento/:id` | WA-70 | cenário a executar em HML (não é aceite) |
| Atendimento WhatsApp | Kanban estreito com rolagem horizontal e arquivados | `/whatsapp` | WA-71 | cenário a executar em HML (não é aceite) |
| Atendimento WhatsApp | Gaveta de contatos como modal no mobile e Esc | `/whatsapp/admin/contatos` | WA-72 | cenário a executar em HML (não é aceite) |
| Atendimento WhatsApp | Movimento reduzido e tema escuro | `/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/admin/contatos` | WA-73 | cenário a executar em HML (não é aceite) |
| Atendimento WhatsApp | Marcar como visto e reabrir (F6) | `/whatsapp`, `/whatsapp/atendimento/:id` | WA-74 | cenário derivado do código (teste backend/spec) |
| Atendimento WhatsApp | Histórico com eventos de finalização e arquivamento (F4) | `/whatsapp/atendimento/:id` (aba Histórico) | WA-75 | cenário derivado do código (teste backend) |
| Atendimento WhatsApp | Grupo por solicitante e um atendimento ativo por grupo (F5) | `/whatsapp/atendimento/:id` | WA-76 | cenário derivado do código (teste backend) |
| Atendimento WhatsApp | Abas Resumo/IA/Ações/Histórico estilizadas e acessíveis (F1) | `/whatsapp/atendimento/:id` | WA-77 | cenário derivado do código (build) |
| Atendimento WhatsApp | Áudio compacto "AUDIO" com play e velocidades (F7) | `/whatsapp/atendimento/:id` | WA-78 | cenário derivado do código (build) |
| Atendimento WhatsApp | Nome do grupo como título do card (Feature 2) | `/whatsapp`, `/whatsapp/minhas-conversas` | WA-79 | cenário derivado do código (teste backend/spec) |
| Atendimento WhatsApp | Ver integrantes do grupo e atualizar a lista (Feature 5) | `/whatsapp/atendimento/:id` (modal Integrantes) | WA-80 | cenário derivado do código (teste backend) |
| Atendimento WhatsApp | SLA visível na fila e no atendimento (Feature 4) | `/whatsapp`, `/whatsapp/atendimento/:id` | WA-81 | cenário derivado do código (sem teste automatizado) |
| Atendimento WhatsApp | Balões e avatar no layout do WhatsApp Web (Feature 3) | `/whatsapp/atendimento/:id` | WA-82 | cenário derivado do código (build) |
| Atendimento WhatsApp | Recorte de status em "Minhas Conversas" (Fase 5) | `/whatsapp/minhas-conversas` | WA-83 | cenário derivado do código |
| Atendimento WhatsApp | Protocolo visível e busca exata no Kanban (Fases 0/1) | `/whatsapp`, `/whatsapp/minhas-conversas` | WA-84 | cenário derivado do código (teste backend/spec) |
| Atendimento WhatsApp | Telefone do contato (Fases 0/1) | `/whatsapp`, `/whatsapp/atendimento/:id` | WA-85 | cenário derivado do código (teste backend/spec) |
| Atendimento WhatsApp | Resolvido finaliza e arquiva sozinho após a janela, exceto grupo (Fase 2) | `/whatsapp` | WA-86 | cenário derivado do código (teste backend) |
| Atendimento WhatsApp | Foto do contato e avatar com fallback (Fase 3) | `/whatsapp`, `/whatsapp/atendimento/:id`, `/whatsapp/minhas-conversas` | WA-87 | cenário derivado do código (teste backend/spec) |
| Atendimento WhatsApp | Métricas por etiqueta e filtro de etiqueta (Fase 4) | `/whatsapp/metricas` (aba Geral) | WA-88 | cenário derivado do código (teste backend) |
| Atendimento WhatsApp | Cadastrar/editar número exigindo setor ativo e recusando setor inativo/inexistente | `/whatsapp/numeros` | WA-89 | cenário derivado do código (teste backend) |
| Atendimento WhatsApp | UX da tela de números: filtros na URL, modal, menu de ações e exclusão em duas etapas | `/whatsapp/numeros` | WA-90 | cenário derivado do código (spec de `tempoRelativo`) |