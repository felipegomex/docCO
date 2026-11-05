---
id: sec.20-contagem-final
titulo: "Contagem final"
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
tokens_estimados: 1439
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# Contagem final

| Grupo solicitado | Histórias ativas |
|---|---:|
| Login/Base | 5 |
| Conhecimento/Ferramentas | 11 |
| Implantação | 8 |
| Kanban Geral | 3 |
| Database | 7 |
| Empresa/Agenda | 5 |
| Acessos Empresas | 2 |
| Dashboard | 7 |
| Admin/Kanban F | 3 |
| Chat/Conhecimento | 2 |
| Ideias, Melhorias e Erros | 6 |
| Sistema (RBAC e operadores) | 6 |
| Navegação/RBAC (2026-10-22) | 5 |
| Atendimento WhatsApp | 60 |
| **Total** | **130** |

- **103** histórias estão como `cenário documentado` (incluindo as **5** do RBAC documentadas, as **5** de navegação/visões gerais, RB-01 a RB-05 e NAV-01 a NAV-05, as **7** da Fase 06, WA-44 a WA-50, as **7** da Fase 07, WA-51 a WA-57, as **5** da Fase 08, WA-58 a WA-62, as **5** da Fase 09, WA-63 a WA-67, as **2** das Fases 0/1 do plano, WA-84 e WA-85, a **Fase 3**, WA-87, e a **Fase 4**, WA-88); **10** como `validado em teste` (LB-04 e IM-06 pela cobertura unitária existente; **WA-01b** por `WhatsAppMensagemSistemaTests`; **KG-01, KG-02 e a regra de matriz de função** pela suíte backend de `KanbanFuncaoTests`; **WA-16 e WA-17** por `WhatsAppStatusSessaoTests` e `WhatsAppFilaRecuperacaoTests`; **RB-06** por `PermissoesCargoHierarquiaTests` e **ID-06** por `IdeiasCiaaLeitorTests`). As **6 histórias da Fase 10 (WA-68 a WA-73)** estão como **`cenário a executar em HML` (não é aceite)** — os estados e a responsividade foram implementados e têm spec, mas a matriz visual não foi executada. Isso não representa execução das suítes nesta revisão. As **6 histórias do módulo de Ideias (ID-01 a ID-06)**: ID-01 a ID-05 foram verificadas por `ng build` (dev e produção, ambos verdes) e pela suíte backend filtrada do módulo (**181 verdes**, incluindo os 5 testes da exclusão); **ID-06** tem cobertura backend em `IdeiasCiaaLeitorTests` (**10** casos) — **não há teste de frontend cobrindo elas**, e o módulo ainda não foi exercitado contra a API em ambiente real. As **19 histórias do atendimento WhatsApp (WA-01 a WA-19)** foram derivadas do código: o módulo está **publicado em HML e PROD** e o **piloto foi executado** em 2026-10-21 (schema aplicado, gateway no ar, número pareado, mensagens reais recebidas, primeira resposta assumindo o atendimento, resolução e reabertura; o gateway ficou **sem sessões** após o reset de 2026-10-27) — **WA-01, WA-02, WA-05 e WA-06** foram exercitadas nesse piloto; as demais seguem **não executadas** como QA. **WA-11 a WA-15** cobrem os **cinco defeitos** encontrados no piloto e corrigidos no mesmo dia (ver [`troubleshooting.md`](../../operacao/troubleshooting.md) §19.9): quatro deles têm **cobertura unitária** no backend (`OpenWaBaseUrlTests`, `WhatsAppQueryTraducaoTests`, `WhatsAppNumeroServiceTests`) e o quinto é detectado pelo `smoke.ps1`. **WA-16 a WA-19** cobrem os **três defeitos novos** (§19.10): WA-16 e WA-17 têm **cobertura unitária**; WA-18 e WA-19 **não** têm teste automatizado.
- O **backend** tem um projeto de testes (`backend/Central_BackEnd.Tests`, **921 casos** na medição local de 2026-10-27, 0 falhas): além dos cenários de Acessos, cifra, senha e Kanban Geral, a suíte do módulo de Ideias (`IdeiasAutorizacaoTests`, `IdeiasCicloDeVidaTests`, `IdeiasKanbanTests`, `IdeiasEspecificacaoTests`, `IdeiasSchemaTests`, `IdeiasContratoJsonTests`, `IdeiasSeedTests`, `IdeiasCompatibilidadeSqlServerTests`, `IdeiasAnaliseIaTests` e, desde 2026-10-27, `IdeiasCiaaLeitorTests`) cobre autorização, ciclo de vida, leitura do CIAA e, desde 2026-10-12, a exclusão administrativa — a suíte do chat foi removida no `cb36ad3` (2026-11-03). `PermissoesCargoHierarquiaTests` (**8 casos**) cobre a hierarquia da matriz. A Fase 07 acrescentou **15 casos** em `WhatsAppKanbanServiceDeskTests`; a Fase 08 acrescentou **16 casos** em `WhatsAppAdminContatoTests`; a Fase 09 acrescentou **13 casos** em `WhatsAppRealtimeTests`; as correções de 2026-10-26 acrescentaram **4 casos** (2 em `WhatsAppQueryTraducaoTests`, 1 em `WhatsAppAdminContatoTests` e 1 em `WhatsAppEtiquetaTests`) e as features F4/F5/F6 acrescentaram **12 casos** (`WhatsAppFinalizacaoTests` 2, `WhatsAppVistoTests` 5 e `WhatsAppGrupoAtendimentoTests` 5); o filtro de avisos de sistema do WhatsApp tem **4 casos** em `WhatsAppMensagemSistemaTests`; a **Fase 6** acrescentou **12 casos** (`WhatsAppProtocoloTests` 3, `WhatsAppFinalizacaoAutomaticaTests` 4, `WhatsAppFotoContatoTests` 3 e `WhatsAppMetricasEtiquetaTests` 2). O inventário atual do frontend tem **177 casos** (171 executados + 6 não executados nesta revisão: 2 de Visto, 3 do avatar e 1 de busca por protocolo/telefone), incluindo os specs do chat (removidos no `cb36ad3`), a permissão do `revisorGuard`, os 12 casos de formulário de Tarefa, os **13 novos** da Fase 07 (card do Kanban, página e `whatsapp.utils`), os **14 novos** da Fase 08 (hub administrativo, gestão de contatos e o item "Admin" do menu), os **18 novos** da Fase 09 (7 no `WhatsAppRealtimeService`, 6 no atendimento em tempo real, 3 em Minhas Conversas e 2 no Kanban) e os **13 novos** da Fase 10 (estados e responsividade: atendimento, gestão de contatos, Kanban e histórico do cliente); execução de 2026-10-26: **171 verdes**. **`karma.conf.js` existe e o pipeline executa `ng test`** (`validate.ps1`, passo 6/7); os casos de frontend permanecem os medidos em 2026-10-26 (não reexecutados nesta revisão). Itens da seção de regressão/histórico não entram no total.

