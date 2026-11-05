---
id: regras-negocio.02
titulo: "Legenda de situação"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 888
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Legenda de situação

- **Implementado**: regra executada pelo backend no código atual.
- **Parcial**: há implementação, mas com limite, inconsistência ou autorização incompleta.
- **Planejado/recomendado**: não foi localizado como regra executada; é uma recomendação deste documento, não um contrato.

| Regra solicitada | Situação |
|---|---|
| autenticação e refresh | **Implementado**, com proteção de transporte parcial |
| atraso de tarefa | **Implementado** com `DataEntrega ?? DataPrevisao`, em regra **única** compartilhada com projeto e etapa |
| data de conclusão real da tarefa | **Implementado**; o cliente **não** informa a data — o servidor a deriva do status/coluna |
| bloqueios | **Parcial**; viraram atributo do card (`tbbloqueio`) e a coluna BLOQUEADO foi desativada |
| apontamentos | **Parcial** |
| sincronização tarefa–agenda | **Parcial** |
| etapas | **Implementado**; a etapa do projeto é obrigatória na tarefa com projeto, com fórmulas de progresso concorrentes |
| Kanban Geral (função, permissão por função, bloqueio, etapas fixas) | **Implementado**; permissão por função em `FuncaoAuthorizationService` (cargo `GESTOR_MASTER` + `tbgestor`), leitura restrita à própria função/geridas e escrita na própria função ou nas geridas |
| propriedade/permissões | **Parcial** |
| conflitos de agenda | **Parcial** |
| código PRJ global | **Parcial** |
| dados sensíveis | **Parcial** (escritas novas de Acessos cifradas com AES; linhas antigas dependem de backfill; `tboperador.SENHA` em texto claro) |
| RBAC (cargo, permissões, policies e gestão de operadores) | **Implementado**; motor único com precedência (bypass Master > negativa individual > positiva individual > cargo > negado), escopo `Proprio`/`Funcao`/`Tudo`, bypass `GESTOR_MASTER`, policies `perm:<recurso>:<acao>` e auditoria de 403. A edição da matriz segue a hierarquia `CARGO_Nivel`/`CARGO_FuncaoId` (Master em tudo, exceto o próprio Master; Tech Lead só no setor e abaixo do nível) |
| módulo de atendimento WhatsApp | **Implementado**; a primeira resposta atribui o atendimento, o estado "Aguardando cliente" mantém o responsável e o tempo em espera é descontado das métricas. **Publicado na homologação em 2026-10-21** (schema aplicado em `dbBUSINESS_HML`, gateway OpenWA no ar na VM `192.168.2.253` e webhook validado) e **com o piloto em operação**: número pareado e mensagens reais entrando no Kanban. O módulo é **ligado por variável de ambiente** no servidor; a **produção (`.254`) foi publicada em 2026-10-26**. As **Fases 0/1 do plano (2026-10-27)** acrescentaram protocolo computado, busca/exibição por protocolo e telefone do contato; a **Fase 2** acrescentou a **finalização automática** de Resolvidos (job de 1 min, exceto grupos), que **finaliza e arquiva** na mesma rodada. A **Fase 3** acrescentou a **foto do contato** (proxy autenticado com TTL/cache) e o telefone normalizado na entrada e nas novas conversas |
| compatibilidade do banco legado (compat level 100) | **Implementado** como restrição permanente de escrita de LINQ |
| regras adicionais planejadas | **Não confirmadas no código** |

Nenhuma regra de negócio adicional “planejada” foi confirmada no código. Os itens marcados como recomendados não devem ser tratados como funcionalidade existente.

