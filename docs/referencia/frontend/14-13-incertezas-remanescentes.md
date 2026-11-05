---
id: sec.14-13-incertezas-remanescentes
titulo: "13. Incertezas remanescentes"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas, componentes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 637
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/frontend.md](../frontend.md).

# 13. Incertezas remanescentes

- O `ng build` foi executado em 2026-10-26 (verde, com avisos de budget já existentes). A **matriz visual manual** (navegador) não foi executada; o documento comprova a configuração estática e os testes automatizados, não a renderização integrada.
- A suíte frontend tinha **171 casos**, entre eles 10 no mascote `app-jota-3d` (`jota-3d.component.spec.ts`), 12 no formulário de Tarefa, dois no `revisorGuard`, três no contrato HTTP do Chat, **13 acrescentados pela Fase 07** (card do Kanban, página e `whatsapp.utils`), **14 acrescentados pela Fase 08** (hub administrativo, gestão de contatos e o item "Admin" do menu), **18 acrescentados pela Fase 09** (7 no `WhatsAppRealtimeService`, 6 no atendimento em tempo real, 3 em Minhas Conversas e 2 no Kanban) e **13 acrescentados pela Fase 10** (estados e responsividade: atendimento, gestão de contatos, Kanban e histórico do cliente). As features F1/F4/F5/F6/F7 acrescentaram **2 casos** em `whatsapp.utils.spec.ts` (Visto) e a **Fase 6** acrescentou **4** (3 em `contato-avatar.component.spec.ts` e 1 em `whatsapp.utils.spec.ts`, busca por protocolo/telefone), levando o inventário a **177** — **não executado** nesta revisão (Karma local não foi rodado). Execução anterior (2026-10-26): **171 SUCCESS**. A classificação por história está em [`qa-historias.md`](../qa-historias.md).
- O estado do ambiente publicado e das migrações de banco não foi verificado.
- A disponibilidade e os resultados reais do Database Explorer e AnythingLLM dependem da configuração e dos serviços externos em execução.
- A divergência entre as constantes de versão (`package.json`/`APP_VERSION` em 0.7.0 e `APP_CONFIG.versao` em 0.8.0) foi resolvida nesta iteração: `APP_VERSION = '1.0.0'` (`app-version.ts`) e `APP_CONFIG.versao = '1.0.0'` (`app-config.ts`) convergem para o SemVer `1.0.0`, publicado via `version.json` (`BuildInfoService`, fallback `APP_VERSION`) e exibido no Guia do Sistema.
- A intenção de produto para o `AdminDashboardComponent` (removido em 2026-10-20, Fase 0) e para o endpoint `GET /api/v1/admin/dashboard`, que segue sem tela, não está registrada; não foi presumida uma correção.
- O backend expõe `GET /auth/me`, mas o frontend atual não o consome; o Chat não oferece API de histórico de conversa. A rota `/api/rag-proxy` não é mais ativa.


