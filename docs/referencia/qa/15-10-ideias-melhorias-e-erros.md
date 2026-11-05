---
id: sec.15-10-ideias-melhorias-e-erros
titulo: "10. Ideias, Melhorias e Erros"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 2677
ficha: docs/modulos/ideias.md
---
> Ficha: [Ideias, Melhorias e Erros](../../modulos/ideias.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 10. Ideias, Melhorias e Erros

> Módulo **em construção**. As telas existem e consomem a API real, mas o módulo **ainda não foi publicado** e os 3 exemplos de demonstração (fase F13) não foram semeados. As ações de fluxo (aprovar, analisar, especificação e vínculo) já têm endpoint no backend.

### ID-01 - Listar e filtrar as solicitações

- **Objetivo:** ver as solicitações às quais o operador tem direito — apenas as próprias, ou todas da sua função se for gestor — com filtros e paginação.
- **Pré-condições:** operador autenticado. Para ver mais que o próprio, é preciso estar em `tbgestor` (`GesoresController` só permite cadastrar gestor a administrador).
- **Passos:** 1. Abrir `/ideias/lista` — ou o item **Ideias e Melhorias** na sidebar do CIAA/Implantação, ou **Sistema → Ideias, Melhorias e Erros**, ou a busca global do header. 2. Se o operador não administra nada, a tela mostra o aviso que o backend devolveu em `minhas-permissoes`, em vez de uma lista vazia sem explicação. 3. Pesquisar por título, filtrar por tipo, status, prioridade e período, marcar **Somente minhas** e alternar "Arquivadas". 4. Abrir uma solicitação pelo título.
- **Colunas:** título, tipo, status, prioridade, criada em, atualizada, e **criador — visível para gestor e para o leitor do setor CIAA** (`podeVerCriador` em `minhas-permissoes`). As prioridades aparecem separadas: `Sugerida:` (do criador) e `Oficial:` (do gestor, com bandeira), para não ler a sugestão como decisão.
- **Estados:** "sem resultado" (há filtro e 0 hits, oferece **limpar filtros**) é diferente de "lista vazia" (sem filtro e 0 hits, oferece **criar a primeira**). Erro de comunicação oferece "Tentar de novo"; sessão expirada pede novo login.
- **Restrições:** a API aceita `prioridade` e `somenteMinhas` em `IdeiaFiltro`; prioridade filtra por `PrioridadeOficial ?? PrioridadeSugerida`. O `IdeiasService` envia os dois parâmetros, e a lista tem o select **"Filtrar por prioridade"** ("Todas as prioridades" + Baixa/Média/Alta) e o checkbox **"Somente minhas"** ao lado de "Arquivadas"; ambos entram em `temFiltro` e em `limparFiltros`.

### ID-02 - Registrar uma solicitação

- **Objetivo:** criar uma ideia, melhoria ou relato de erro, com o formulário adaptado ao tipo.
- **Pré-condições:** operador autenticado.
- **Passos:** 1. Abrir `/ideias/nova` ou o botão "Nova solicitação". 2. Escolher o tipo entre os 3 cards — o formulário troca os campos na hora. 3. Preencher. 4. Enviar; o sucesso leva direto ao detalhe.
- **Campos por tipo:** **Ideia** e **Melhoria** pedem problema/oportunidade + solução proposta, com benefício esperado opcional. **Relato de erro** pede comportamento observado + esperado, com passos, impacto, frequência e URLs opcionais. Os obrigatórios espelham literalmente o backend.
- **⚠️ Atenção na troca de tipo:** o registro guarda campos de Ideia **e** de Relato no mesmo lugar. Ao trocar de tipo com texto preenchido, a tela abre uma confirmação **listando o que será descartado**. Não há descarte silencioso.
- **Nota de rótulo:** "Melhoria" não tem campo "comportamento atual" próprio na API — esse texto é gravado em `Problema`, e o rótulo muda por tipo sobre a mesma coluna.
- **Erro:** se a gravação falhar, o texto digitado **permanece** na tela; só é limpo após confirmação do servidor.

### ID-03 - Acompanhar e responder uma solicitação

- **Objetivo:** ver a solicitação inteira — dados, timeline e ações — e agir conforme a fase atual.
- **Pré-condições:** ser o criador ou o gestor da função do criador. Sem acesso de leitura, a API devolve **404** e a tela mostra "Solicitação não encontrada" — indistinguível de registro inexistente, por desenho. Ação de escrita sem permissão devolve **403** e grava o evento `NEGADO` na auditoria.
- **Passos:** 1. Abrir `/ideias/{id}`. 2. Ler o cabeçalho: tipo, status, prioridades, tarefa vinculada. 3. Comentar na conversa. 4. Se o gestor pedir esclarecimentos, aparece um aviso dedicado e o campo de resposta vira a mensagem **e** devolve a solicitação para "Em discussão". 5. Gestor aprova no modal de decisão, escolhendo a função e, opcionalmente, o responsável pela tarefa.
- **Timeline:** a seção chama **"Conversa e historico"** e explica que reúne mensagens de quem abriu e da gestão em ordem cronológica. Mensagens e decisões ficam na mesma lista; cada item traz `ehCriador` e o selo **"Criador"** aparece quando o autor é quem abriu a solicitação. **Decisões e transições** têm trilho colorido à esquerda e ícone próprio, para não se confundirem com comentário comum. O texto da decisão vem pronto do servidor (`TransicoesStatusIdeia.Rotulo`), não remontado no cliente.
- **Edição:** o botão "Editar" só aparece para o **criador**, e só em `Recebida` e `AguardandoEsclarecimentos` — é a matriz do backend que decide. Em `Em Discussão` o criador ainda conversa, mas não reescreve o pedido.
- **Ações de fluxo:** "Aprovar", "Desvincular do Kanban", "Analisar com IA" e "Especificação técnica" têm endpoint no backend (`/aprovar`, `/vinculo-kanban`, `/analises` e `/especificacao`). A visibilidade segue a matriz de papéis; clique sem permissão devolve 403 auditado. Sem coluna compatível no Kanban, aprovar falha como erro de negócio.
- **Aprovação com responsável (F7):** no modal de decisão, o select **"Responsavel pela tarefa"** (carregado de `GET /api/v1/agenda/operadores`) traz a opção **"Sem responsavel (alguem da funcao assume)"**; os campos de função e responsável só aparecem quando `Aprovar` é permitido, e o request envia `responsavelId`. O servidor exige operador ativo (`SE_ATIVO='S'`); se a função informada divergir do `FuncaoId` do responsável, devolve **400** ("O responsavel escolhido nao pertence a funcao responsavel informada."). Sem função informada, usa a do responsável; sem escolha, a tarefa nasce sem atribuição. A auditoria `APROVACAO` grava `ResponsavelId`.
- **Read-only:** em `Implementada` a tela inteira vira leitura, com aviso explícito. A conversa não aceita novas mensagens.
- **Recusa e arquivamento** exigem justificativa; o motivo da recusa fica destacado no topo da tela e preservado no histórico mesmo após reabertura.

### ID-04 - Configurar gestores por setor (tela GESTORES)

- **Objetivo:** dizer quem administra as solicitações de cada setor, sem editar tabela.
- **Pré-condições:** perfil `Administrador` para gerenciar. A tela é alcançada pelo dropdown do header (**Perfil → Administração → Gestores**, rota `/gestores-ideias`) — não há item na sidebar. O backend revalida por claim: sem admin, `GET /gestores` devolve 403 e a tela vira **somente leitura** (funções sempre visíveis, aviso "Modo somente leitura" e botões de cadastrar/definir/excluir ocultos); outros erros mantêm o erro bloqueante.
- **Passos:** 1. Abrir GESTORES. 2. Conferir a tabela de setores — cada linha vem de `GET /funcoes?apenasAtivos=true` (`tbfuncao`): setor, classificação, quantidade de funcionários ativos (`GET /operadores?funcaoId=&apenasAtivos=true`, `SE_ATIVO='S'`) e gestor(es). 3. Clicar em **Funcionários** para ver os operadores ativos daquela função. 4. Clicar em **Definir gestor**, buscar o operador e cadastrar — qualquer operador ativo serve (o backend não exige que seja da função); os da função vêm marcados "do setor". 5. Para o gestor geral, usar **Cadastrar gestor geral** no topo ou o checkbox no modal. 6. Excluir pelo botão no chip do gestor (ou no aviso do gestor geral), com confirmação.
- **Setor novo aparece sozinho:** função cadastrada em `tbfuncao` e ativa entra na lista na próxima carga — nada é fixo no front (ex.: a futura "CIAA - Centro de IA e Automação"). Função inativa não é oferecida como setor.
- **⚠️ Se nenhum gestor ativo existir,** a tela mostra um aviso explicando que ninguém administra o módulo e que os operadores só veem as próprias solicitações. É a causa raiz de uma listagem vazia, dita antes de ela acontecer.
- **Limitação:** o backend **não expõe troca de função** de um gestor já cadastrado. Para trocar, exclua e cadastre outro.
- **Regra do servidor (2026-10-13):** no máximo **um gestor ativo por função** — cadastrar outro substitui o atual (a resposta traz `substituiuOperadorId/Nome`); remover é **exclusão** (`DELETE`), e excluir o último gestor geral ativo é recusado com `400`.

### ID-05 - Arquivar uma solicitação (exclusão lógica, administrador ou gestor)

- **Objetivo:** tirar da lista ativa uma solicitação criada por engano ou inviável **sem perder o histórico**. A exclusão é **lógica** (arquivamento): não há delete físico.
- **Pré-condições:** perfil `Administrador` **ou gestor da ideia** (geral ou da função do criador). No detalhe o botão **Arquivar** (ícone `bi-archive`, com `title` avisando que nada é apagado) aparece para admin ou `papel === 'Gestor'` (ao lado de "Desvincular do Kanban"); na listagem, só para admin. Isso é orientação de tela: **quem barra é o servidor** — `DELETE /api/v1/ideias/{id}` devolve 403 para quem não é nem um nem outro (autor sem gestão não arquiva), grava `NEGADO` na auditoria e responde 404 quando a solicitação não existe.
- **Passos:** 1. Em `/ideias/lista`, clicar em **Excluir** na linha da solicitação (ou abrir `/ideias/{id}` e usar **Arquivar**). 2. Confirmar o diálogo: "Arquivar a solicitacao #...? Ela sai da lista ativa, mas mensagens, historico, especificacoes e analises continuam preservados." 3. A lista recarrega (no detalhe, a tela volta para `/ideias/lista`) com toast "Solicitacao #... arquivada".
- **Resultado esperado:** 204 do servidor. A solicitação continua no banco com `Arquivada = true`, `DataArquivamento`, `MotivoArquivamento = "Exclusao administrativa"` e `UsuarioAlteracao/DataAlteracao`; a auditoria grava `ARQUIVAR`. Mensagens, especificações, análises e histórico **permanecem**; a tarefa do Kanban vinculada também **não** é apagada (fica órfã no quadro, porque pode ter apontamento de horas).
- **Distinção importante:** excluir e arquivar agora são ambos **lógicos**. `DELETE /ideias/{id}` é a exclusão administrativa (admin ou gestor, motivo fixo, botão **Arquivar**); `POST /ideias/{id}/arquivar` é o formulário **"Arquivar solicitacao"**, exige motivo informado e é exclusivo do gestor. Nenhum dos dois apaga satélites.
- **Automação:** `IdeiasCicloDeVidaTests#Excluir_arquiva_a_solicitacao_e_preserva_os_registros_satelites`.

### ID-06 - Leitor do setor CIAA

- **Objetivo:** confirmar que todo operador do CIAA lê as solicitações criadas por colegas do mesmo setor, sem ganhar escrita.
- **Pré-condições:** operador e criador com `tboperador.FUNCAO_ID = 5` (`IdeiasPermissaoService.FuncaoCiaa`); o criador **não** é o operador; `Permissoes_Matriz_Idempotente.sql` aplicado (linha `aba.ciaa.ideias` com `PEC_FuncaoId = 5`).
- **Passos:** 1. Entrar com o operador do CIAA e abrir `/ideias/lista`. 2. Conferir que a solicitação do colega aparece e que a coluna **Criador** está visível. 3. Abrir o detalhe. 4. Tentar comentar, editar e mudar o status. 5. Repetir com uma solicitação de outro setor.
- **Resultado esperado:** a lista inclui as solicitações do CIAA (além das próprias e das funções geridas), e o detalhe abre com `papel = Leitor`. A matriz de papeis concede **somente `Ler`** ao leitor: não há comentário, edição, resposta a esclarecimento, transição nem aprovação — a API recusa e a tela não oferece as ações. Solicitação de outro setor continua invisível (`404` no detalhe e ausente da lista). `SomenteMinhas` volta a mostrar apenas as do próprio operador.
- **Estado atual/bloqueio:** o leitor depende de a linha por função `aba.ciaa.ideias` estar ativa em `tbperm_cargo`; o card/menu usa essa permissão, mas a barreira real é a matriz de papeis do backend. Não há teste de tela.
- **Rotas/endpoints/tabelas:** `/ideias/lista`, `/ideias/{id}`; `GET /api/v1/ideias`, `GET /api/v1/ideias/{id}`, `GET /api/v1/ideias/minhas-permissoes` (`ehLeitor`, `podeVerCriador`); `tbideia`, `tbperm_cargo`, `tbpermissao_recurso`, `tboperador`.
- **Status/automação:** **validado em teste** no backend por `IdeiasCiaaLeitorTests` (**12** casos: leitura, recusa de escrita, isolamento de setor, lista, `SomenteMinhas`, filtro de prioridade, detalhe e nome do criador); **não há teste de tela**.

