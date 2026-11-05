---
id: 04-3-controllers-e-endpoints.05
titulo: "3.5 Projetos e cards de etapa"
tipo: referencia
area: backend
sistema: central
tags: [backend, controllers, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 728
ficha: docs/modulos/kanban.md
---
> Parte de [3. Controllers e endpoints](./INDEX.md). Ficha: [Kanban Geral](../../../modulos/kanban.md).

### 3.5 Projetos e cards de etapa

Todos os endpoints usam JWT. O controller aplica `validacao`; a listagem de clientes também declara a política `leitura` no método.

| Método e caminho | Corpo/parâmetros | Autorização/efeito |
|---|---|---|
| `GET /api/v1/implantacao/projetos` | Filtros de tipo, status, cliente, responsável, busca, perfil e `funcaoId` | Lista até 500 Projetos; cada item traz `atrasado` (decisão do backend, ver 4.2.2). Fora do Master, o serviço limita às funções de `FuncoesDeLeitura` e devolve lista vazia para setor não permitido |
| `GET /api/v1/implantacao/projetos/com-etapas` | Mesmos filtros | Lista Projeto com cards; pode inicializar cards ausentes |
| `GET /api/v1/implantacao/projetos/{id}` | Sem corpo | Detalhe e contadores de Tarefas; `PodeLerAsync` → `403` para projeto de setor não permitido |
| `GET /api/v1/implantacao/projetos/etapas-padrao` | Sem corpo | Nove nomes padrão |
| `GET /api/v1/implantacao/projetos/{id}/etapas` | Sem corpo | Cards e contadores dinâmicos; `PodeLerAsync` → `403` |
| `GET /api/v1/implantacao/projetos/{id}/etapas/{ordem}` | Sem corpo | Card com checklist, documentos, histórico e comentários; `PodeLerAsync` → `403` |
| `GET /api/v1/implantacao/projetos/proximo-codigo` | Sem corpo | Próximo código `PRJ-####` calculado |
| `GET /api/v1/implantacao/projetos/clientes` | Sem corpo | Clientes ativos de `tbcliente` |
| `POST /api/v1/implantacao/projetos` | `ProjetoCriarRequest` (com `FuncaoId?`) | Cria Projeto e nove cards; `FuncaoId` opcional precisa existir/estar ativo e passar em `PodeCriarFuncao`; sem ele, herda a função do criador. Etapa anterior pode nascer concluída |
| `PUT /api/v1/implantacao/projetos/{id}` | `ProjetoAtualizarRequest` | Atualiza e audita |
| `PUT /api/v1/implantacao/projetos/{id}/etapas/{ordem}` | `ProjetoEtapaAtualizarRequest` | Atualiza checklist/estado e recalcula Projeto |
| `POST /api/v1/implantacao/projetos/{id}/etapas/retornar` | `ProjetoEtapaRetornoRequest` | Retorna a jornada e reseta cards posteriores |
| `POST .../{id}/etapas/{ordem}/checklist` | `ProjetoEtapaChecklistItemRequest` | Acrescenta item |
| `POST .../{id}/etapas/{ordem}/documentos` | `ProjetoEtapaDocumentoRequest` | Acrescenta documento e histórico; recebe URL, sem upload binário |
| `POST .../{id}/etapas/{ordem}/comentarios` | `ProjetoEtapaComentarioRequest` | Acrescenta comentário |
| `DELETE /api/v1/implantacao/projetos/{id}` | Sem corpo | Exclui Projeto e cards pelo cascade configurado |

Os contratos estão em [`ProjetoDtos.cs`](../../../../backend/Central_BackEnd/Dtos/Implantacao/ProjetoDtos.cs).

