---
id: regras-negocio.23
titulo: "Divergências e correções sugeridas"
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
tokens_estimados: 729
ficha: docs/modulos/README.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## Divergências e correções sugeridas

As correções abaixo não foram implementadas; são pontos de atenção verificados no código.

| Achado | Correção sugerida |
|---|---|
| `cc_refresh` com `Secure=false` | habilitar `Secure` em HTTPS e validar o comportamento local sem expor o cookie em trânsito. |
| Credenciais antigas de Acessos Empresas sem `v1:` | concluir o backfill das linhas antigas e avaliar rotação da chave; a escrita nova já usa `CifradorAesGcm`. Ver [`banco-dados.md`](../../arquitetura/banco-dados.md). |
| `tboperador.SENHA` é `varchar(10)` em texto claro e está fora do alcance da nova cifra | migrar o login para hash adaptativo (sal + algoritmo robusto) em projeto separado, com plano de migração das senhas existentes. |
| Refresh, ticket de acesso e defesa contra força bruta têm controles em memória | avaliar estado compartilhado, rotação de chaves e impacto entre instâncias. |
| `F` é papel especial no `hasRole` do frontend, mas não é papel JWT emitido | alinhar a autorização de frontend e backend antes de tratar `F` como papel garantido. |
| Identidade pode ser aceita pelo payload | derivar autor, criador e usuário de alteração do claim, ignorando campos do cliente. |
| Tarefa–agenda não tem vínculo/regra de concorrência | adicionar idempotência, transação e teste de corrida, ou declarar a sincronização como apenas melhor esforço. |
| Agenda persiste `Visibilidade`, mas leituras não a filtram | aplicar a visibilidade e a autorização de leitura no serviço ou registrar a exposição como limitação aceita. |
| Alerta “Bloqueadas” não usa `Bloqueada` | corrigir a consulta ou renomear o alerta. |
| `ClienteObrigatorio` não é validado | implementar a regra ou retirar o campo da descrição de negócio. |
| Atraso de faturamento é inferido do dia do mês, não de data real | o legado não guarda a data; o modelo considera o mês corrente vencido depois do dia configurado. Registrar a limitação no produto ou criar uma data de faturamento controlada pelo JCA. |
| Configuração de desenvolvimento contém segredos potenciais | externalizar, rotacionar e não redistribuir o arquivo preenchido. |

**Planejados/recomendados:** não há implementação confirmada para propriedade estrita por projeto/tarefa, hash de senha adaptativo, criptografia do `tboperador.SENHA` ou proteção por papel no Database Explorer. A cifra das credenciais de Acessos Empresas **existe** (AES-256-GCM pronta), mas **não está registrada no DI** — trata-se de desligamento deliberado, não de item ausente. Esses itens só podem ser marcados como planejados após decisão de produto/segurança.

