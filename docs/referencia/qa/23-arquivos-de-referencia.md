---
id: sec.23-arquivos-de-referencia
titulo: "Arquivos de referência"
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
tokens_estimados: 507
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# Arquivos de referência

- [`05-ENDPOINTS.md`](../endpoints.md), [`06-COMPONENTES-FRONTEND.md`](../frontend.md) e [`07-SERVICES-BACKEND.md`](../backend.md) — contratos, componentes e serviços atuais. Roteiros e índices anteriores foram consolidados aqui e removidos em 09/10 (histórico no Git).
- Rotas conferidas em `frontend/src/app/app.routes.ts`, `frontend/src/app/features.routes.ts`, `frontend/src/app/features/dashboard/dashboard.routes.ts`, `frontend/src/app/features/dashboard/comercial/comercial.routes.ts`, `frontend/src/app/features/implantacao/implantacao.routes.ts`, `frontend/src/app/features/database/database.routes.ts` e `frontend/src/app/features/admin/admin.routes.ts`; endpoints conferidos nos controllers do backend.
- **WA-74 a WA-78** foram derivados diretamente do código das features **F1/F4/F5/F6/F7** e têm cobertura automatizada parcial (testes backend `WhatsAppFinalizacaoTests`, `WhatsAppVistoTests` e `WhatsAppGrupoAtendimentoTests`; spec de `whatsapp.utils`); não são roteiros de QA manual executados.
- **WA-84 a WA-88** (Fases 0/1, Fase 2, Fase 3 e Fase 4 do plano) também são derivados do código: protocolo e telefone têm teste de tela parcial; a **finalização automática** é coberta por `WhatsAppFinalizacaoAutomaticaTests` (finaliza + arquiva + eventos) e a configuração da janela por `WhatsAppConfiguracaoTests`. A **foto** tem serviço e proxy implementados, mas **não há teste automatizado** de backend nem spec do avatar nesta revisão. As **métricas por etiqueta** foram confirmadas no `WhatsAppMetricasService`, nos DTOs e na tela, mas **não há teste automatizado** do bloco por etiqueta nesta revisão.

