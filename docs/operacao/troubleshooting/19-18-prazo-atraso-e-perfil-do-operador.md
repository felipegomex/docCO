---
id: sec.19-18-prazo-atraso-e-perfil-do-operador
titulo: "18. Prazo, atraso e perfil do operador"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 913
ficha: docs/modulos/auth-rbac-sistema.md
---
> Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 18. Prazo, atraso e perfil do operador

Duas divergÃªncias de regra que sÃ³ aparecem quando **duas telas** (ou **dois fluxos**) respondem sobre a mesma coisa. Ambas foram corrigidas em 2026-10-21; os sintomas abaixo descrevem o que a versÃ£o anterior produz.

### A aba "Atrasados" e o Dashboard mostram nÃºmeros diferentes

**Sintoma.** A contagem da aba "Atrasados" (`/implantacao/projetos`) nÃ£o bate com o KPI "projetos atrasados" do Dashboard (`/implantacao/dashboard`) para o mesmo conjunto de projetos.

**Causa.** Regras **separadas**: a aba contava **etapa** atrasada (qualquer card com `atrasoDias > 0`) e o Dashboard contava **prazo do projeto** vencido. Os dois nÃºmeros medem coisas diferentes, entÃ£o discordavam â€” a diferenÃ§a nÃ£o era de arredondamento nem de recorte.

**SoluÃ§Ã£o.** A regra Ã© **uma sÃ³** e fica no backend, em [`RegrasPrazo`](../../../backend/Central_BackEnd/Services/Shared/RegrasPrazo.cs):

| Objeto | Regra |
|---|---|
| Projeto | **prazo vencido** (`DataPrevisao`) e nÃ£o concluÃ­do/cancelado |
| Tarefa | prazo (`DataEntrega ?? DataPrevisao`) vencido, **exceto** concluÃ­da dentro do prazo; cancelada nunca conta |
| Etapa | atraso em dias com referÃªncia `DataFimReal` ou hoje |

A decisÃ£o sobe no contrato (`atrasado` no resumo de projeto, no resumo de tarefa e no card do quadro) e as telas consomem o campo. Se os nÃºmeros voltarem a divergir, **o problema nÃ£o Ã© a tela**: confirme que o backend publicado traz o campo `atrasado` (versÃ£o do `BUILD_INFO.GitCommit`) e que o `AtrasoDias` da etapa estÃ¡ sendo recalculado â€” quem recalcula Ã© a **sincronizaÃ§Ã£o por tarefas**, nÃ£o sÃ³ a ediÃ§Ã£o manual. (No quadro do Kanban o card jÃ¡ recebe `atrasado`/`dataConclusao` do backend; o Ãºnico ponto que ainda usa o espelho local Ã© o `TarefaDetalhe` do drawer, cujo DTO nÃ£o expÃµe `atrasado` â€” ver [`frontend.md`](../../referencia/frontend.md) Â§ 4.4.)

### O mesmo operador Ã© admin no Kanban e nÃ£o-admin nos apontamentos

**Sintoma.** O operador passa pelas permissÃµes do Kanban Geral, mas Ã© recusado (`403`) ao editar ou excluir apontamento de outro operador â€” ou o contrÃ¡rio.

**Causa.** `PERFIL_ID` Ã© uma letra em `tboperador`, coluna `varchar(1)` com **cadastro manual**: nada impede `"a"` minÃºsculo ou `" A "` com espaÃ§o. Um serviÃ§o lia a letra **crua** e outro a **normalizava** (trim + maiÃºsculas), e o token carregava a letra como estava no banco. A mesma pessoa recebia duas respostas para a mesma pergunta.

**SoluÃ§Ã£o.** A leitura Ã© Ãºnica, em [`PerfilOperador`](../../../backend/Central_BackEnd/Services/Shared/PerfilOperador.cs): normaliza (trim + maiÃºsculas), decide admin (`SE_ADMIN` ou letra `A`) e produz o rÃ³tulo. O token carrega a claim **`perfil_id` jÃ¡ normalizada**, e o controller de apontamentos reconhece o admin por `IsInRole("Administrador")`, claim `eh_admin` ou `perfil_id = "A"`.

Para confirmar sem expor dado sensÃ­vel: compare a claim `perfil_id` do token com o valor gravado em `tboperador.PERFIL_ID` â€” diferenÃ§a de caixa ou espaÃ§o Ã© a causa. Se o sintoma persistir depois da correÃ§Ã£o, o backend publicado Ã© anterior a ela (confira `BUILD_INFO.GitCommit`); e vale **normalizar o cadastro** em `tboperador`, que hoje aceita qualquer caixa/espaÃ§o.

