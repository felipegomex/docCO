---
id: whatsapp.04
titulo: "Números (`/numeros`)"
tipo: referencia
area: api
sistema: central
tags: [api, whatsapp, endpoints]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers/WhatsApp
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 1413
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Números (`/numeros`)

Leitura (`GET`) é de qualquer atendente: quem responde precisa saber se o número está conectado. **Criar, editar, iniciar, parar, reconectar, ler o QR e remover são de administrador** — o backend responde `403` para os demais.

**Todo número pertence a exatamente um setor ativo da `tbfuncao`.** O cadastro usa `funcaoId` (FK para `tbfuncao.FUNCAO_ID`); a resposta mantém o campo `fila`, mas ele é **derivado** de `tbfuncao.DESCRICAO` — não é mais texto livre nem editável. `ativo` é derivado de `statusNumero` (Ativo).

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/numeros` | atendente | — | `200 List<NumeroWhatsApp>` (sessão, descrição, telefone, `funcaoId`, **`fila` derivada do setor**, `ativo`, `statusNumero`, status do gateway, mensagem de encerramento, atendimentos ativos, última mensagem) |
| `GET` | `/numeros/{id}` | atendente | Path | `200 NumeroWhatsApp`; `404` |
| `POST` | `/numeros` | **administrador** | Body `CriarNumeroRequest`: `descricao` (1–100), **`funcaoId`** (obrigatório — setor ativo da `tbfuncao`), `mensagemEncerramento?`. **`fila` não é aceito** | `201 NumeroWhatsApp`. O **servidor** cria a sessão no gateway (`central-wa-<id>`), **a inicia** e registra o webhook; o cliente não manda id de sessão. `400 { mensagem }` quando o setor está ausente, não existe ou está inativo |
| `PATCH` | `/numeros/{id}` | **administrador** | Body `AtualizarNumeroRequest`: `descricao?`, **`funcaoId?`**, **`statusNumero?`**, **`ativo?`** (atalho booleano: `true` = Ativo, `false` = Desativado; tem precedência sobre `statusNumero`), `mensagemEncerramento?` | `200 NumeroWhatsApp`; não mexe na sessão do gateway. `400 { mensagem }` se o setor informado não existir/estiver inativo (`400` também para descrição vazia ou acima de 100 caracteres) |
| `POST` | `/numeros/{id}/iniciar` | **administrador** | — | `200 NumeroWhatsApp` (status espelhado do gateway) |
| `POST` | `/numeros/{id}/parar` | **administrador** | — | `200 NumeroWhatsApp` |
| `POST` | `/numeros/{id}/reconectar` | **administrador** | — | `200 NumeroWhatsApp`; recria a sessão se ela não existir no gateway, senão para e inicia |
| `GET` | `/numeros/{id}/qr` | **administrador** | — | `200 QrCodeWhatsApp` (QR em data URL + status). **O conteúdo do QR não vai para log** |
| `DELETE` | `/numeros/{id}` | **administrador** | Query `forcar` (default `false`) | `204`; remove **fisicamente** o número e a sessão no gateway (`DELETE /api/sessions/{id}` → `204`, com queda para o `stop` se a rota falhar). **Com histórico e sem `forcar`: `409 { mensagem, codigo: "NUMERO_COM_HISTORICO" }`** - use arquivar ou `?forcar=true`. **Com `forcar=true`: `204`** e a história do número é apagada em cascata (eventos, mensagens, vínculos de etiqueta, atendimentos, conversas, grupos, participantes e fila de webhook da sessão) - **irreversível**. Grupos, participantes e fila da sessão são derivados: saem **sempre**, mesmo sem `forcar` |
| `POST` | `/numeros/{id}/arquivar` | **administrador** | — | `200 NumeroWhatsApp`; transição Desativado → **Arquivado** (remove do quadro operacional, permanece pesquisável); `404` |

**Setor do número.** O `WhatsAppNumeroService.ValidarFuncaoAsync` exige `funcaoId` maior que zero, setor existente em `tbfuncao` e `ATIVO = 1` — as mensagens do `400 { mensagem }` são explícitas ("Selecione o setor responsável pelo número.", "Setor não encontrado.", "O setor selecionado está inativo."). Na edição, a validação só roda quando `funcaoId` vem no corpo. As projeções do Kanban (`AtendimentoService` e `ContatoAdminService`) leem o nome por `Numero.Funcao.Descricao`, e o filtro `fila` de `GET /atendimentos` continua comparando **esse nome** — por isso a tela mantém o campo `fila` na resposta.

O gateway OpenWA v0.25.0 **expõe** rota de exclusão de sessão (`DELETE /api/sessions/{id}`): a sessão é removida de verdade, e não apenas parada. Se o gateway não tiver a rota (ou estiver fora do ar), o serviço cai para o `stop` e a remoção do cadastro acontece de qualquer forma. Sobre o histórico, o contrato é assim:

- **Sem `forcar`** (padrão), o `DELETE` é recusado com **`409 { mensagem, codigo: "NUMERO_COM_HISTORICO" }`** quando o número tem conversas, atendimentos ou mensagens - a alternativa administrativa é **arquivar**. Número **sem** histórico é removido fisicamente. Grupos/participantes do número e a fila de webhook da sessão **não bloqueiam**: são removidos junto em qualquer exclusão.
- **Com `forcar=true`**, o `RemoverAsync` apaga em cascata a história do número numa **única `SaveChanges`** (atômica; as entidades ficam rastreadas e o EF ordena os `DELETE`s por dependência): eventos, mensagens, **vínculos de etiqueta** (`tbwhatsapp_atendimento_etiqueta` - a **definição** em `tbwhatsapp_etiqueta` sobrevive), atendimentos, conversas, **grupos**, **participantes** dos grupos e **fila de webhook da sessão** (`tbwhatsapp_webhook_fila` por `SessaoId`). A FK circular conversa→atendimento é `NO ACTION`, então `Conversa.AtendimentoId` é **anulado em memória antes** de remover os atendimentos. `ExecuteDeleteAsync` foi descartado de propósito: commitaria cada statement isoladamente (um falho no meio deixaria história pela metade) e o provider InMemory não traduz a operação. A auditoria registra a ação como `remover_forcado`.

