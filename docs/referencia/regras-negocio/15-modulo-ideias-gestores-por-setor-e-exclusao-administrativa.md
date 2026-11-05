---
id: regras-negocio.15
titulo: "Módulo Ideias — gestores por setor e exclusão administrativa"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 767
ficha: docs/modulos/ideias.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Ideias, Melhorias e Erros](../../modulos/ideias.md).

## Módulo Ideias — gestores por setor e exclusão administrativa

**Fonte única do escopo: `tbgestor`.** Administrador de sistema (`SE_ADMIN`/`PERFIL_ID='A'`) **não** concede acesso a solicitação nenhuma: quem administra é quem está cadastrado em `tbgestor` (`GES_FuncaoId` = função administrada; NULL = gestor geral, que responde por todas). A função administrada sai do `tboperador.FUNCAO_ID` do **criador** da solicitação. Sem permissão de **leitura**, a API devolve **404** (não revela existência); sem permissão em **ação de escrita**, devolve **403** e registra `NEGADO` na auditoria (`NegarAsync`). Implementação: [`IdeiasPermissaoService.cs`](../../../backend/Central_BackEnd/Services/Ideias/IdeiasPermissaoService.cs) e [`IdeiasController.cs`](../../../backend/Central_BackEnd/Controllers/IdeiasController.cs).

**Leitor do setor CIAA (2026-10-27).** `PapelIdeia.Leitor = 3` cobre o operador e o criador ambos em `FUNCAO_ID = 5` (`IdeiasPermissaoService.FuncaoCiaa`). O leitor entra **somente** na operação `Ler`, em todas as fases: não comenta, não edita, não responde esclarecimento e não transiciona. `EhLeitorCiaaAsync` expõe a condição, e `ListarAsync` inclui as solicitações cujo criador é do CIAA quando o operador também é do CIAA — além das próprias e das funções que ele gerencia. `GET /api/v1/ideias/minhas-permissoes` devolve `ehLeitor` e `podeVerCriador` (gestor **ou** leitor), e o aviso de "não administra" deixa de aparecer para o leitor.

- **Vários gestores por função são válidos** (índice único filtrado é por função+operador); **só um gestor geral ativo** pode existir, e desativar o último é recusado com `400`.
- **O gestor não precisa ser da função que administra.** `GestorService.SalvarAsync` valida apenas operador ativo (`SE_ATIVO='S'`) e função existente. A tela GESTORES reflete isso: oferece qualquer operador ativo no modal, marcando os da própria função como atalho.
- **Tela GESTORES** (rota `/gestores-ideias`, acessada pelo dropdown do header **Perfil → Administração → Gestores**, visível só para `Administrador`): setores vêm de `GET /funcoes?apenasAtivos=true` (`tbfuncao`) e funcionários ativos de `GET /operadores?funcaoId=&apenasAtivos=true`. Função nova cadastrada e ativa no banco aparece sozinha na próxima carga; função inativa não é oferecida como setor.
- **Exclusão administrativa (lógica):** `DELETE /ideias/{id}` **arquiva** a solicitação — grava `Arquivada = true`, `DataArquivamento`, `MotivoArquivamento = "Exclusao administrativa"` e `UsuarioAlteracao/DataAlteracao`, e audita `ARQUIVAR`. Nenhum registro é apagado: mensagens, especificações, análises e histórico permanecem, e a tarefa do Kanban vinculada fica órfã no quadro. Permitida a **administrador ou gestor** (geral ou da função do criador); 403 para os demais, com evento `NEGADO` na auditoria, e 404 quando não existe. É **distinta do arquivamento comum** (`POST /ideias/{id}/arquivar`), que exige motivo informado e é exclusivo do gestor. A matriz de operações do fluxo não é consultada, porque excluir é manutenção do cadastro, não etapa do ciclo de vida.

