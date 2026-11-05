---
id: implantacao.11
titulo: "Limitações de contrato a corrigir (Implantação)"
tipo: referencia
area: api
sistema: central
tags: [api, implantacao, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/Implantacao
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 677
ficha: docs/modulos/kanban.md
---
> Parte de [Endpoints — Controllers/Implantacao](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

## Limitações de contrato a corrigir (Implantação)

As observações abaixo são limitações confirmadas, não regras desejadas:

- [Limitação] `TiposProjetoController` e os métodos de escrita de `ColunasKanbanController` declaram respostas resumidas, mas os serviços retornam DTOs detalhados. O catálogo registra o tipo efetivamente serializado; as assinaturas devem ser alinhadas.
- [Limitação] A exclusão de tipo com projeto vinculado pode gerar `500` por `InvalidOperationException` não tratado; isso é comportamento atual, não uma resposta de negócio desejada.
- [Limitação] O CRUD de `implantacao/etapas-funcao` está exposto e testado no backend, mas **não é exibido no frontend** por decisão de negócio (etapas fixas). Trate essas rotas como contrato de API, não como funcionalidade de tela.
- [Limitação] `GET /implantacao/funcoes` é somente leitura e não tem CRUD por decisão de arquitetura: manter função é tarefa do cadastro de operadores. Não crie um segundo cadastro da mesma unidade.
- [Limitação] Cards com `FUNCAO_ID NULL` — os que estavam nos setores RH e Comercial — **não aparecem em nenhum recorte por função**; a pill "Todas" foi **removida do Kanban Geral** (2026-10-01), então hoje esses cards só voltam ao quadro com uma função atribuída em `tbfuncao`. É consequência direta do mapa de conversão, não um bug de filtro.
- [Limitação] `GET /implantacao/kanban/board` não filtra por perfil e não tem filtro de projeto por perfil; o recorte `?perfil=` é montado no cliente cruzando `board?tipo=tarefas` com `GET /implantacao/tarefas`. O parâmetro é **deep link** e, desde 2026-10-20, **nenhum item de menu o envia** (o "Kanban ADM" saiu do menu por duplicar `/kanban`). Com o recorte ativo, os cards de projeto ficam ocultos.
- [Limitação] A detecção de administrador no frontend do Kanban (`ehGestor`) é um espelho do backend; se a matriz de `FuncaoAuthorizationService` mudar, a UI precisa mudar junto — o backend continua sendo a fonte da verdade.
- [Limitação] O `CardKanban` devolvido traz `atrasado` e `dataConclusao`, e o **cliente mapeia os dois** para o card local do quadro (`CardKanban` → `paraCardQuadro`): no modo geral a métrica "atrasadas" e o rótulo "Vencida" do card consomem a decisão do backend. A lacuna que resta é pontual: o `TarefaDetalhe` do drawer não expõe `atrasado`, então o rótulo "Vencida" **dentro do drawer** usa o espelho local do helper.
