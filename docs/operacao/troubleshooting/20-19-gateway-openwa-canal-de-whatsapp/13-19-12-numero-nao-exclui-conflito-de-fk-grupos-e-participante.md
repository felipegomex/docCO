---
id: 20-19-gateway-openwa-canal-de-whatsapp.13
titulo: "19.12 Número não exclui / conflito de FK (grupos e participantes)"
tipo: operacao
area: diagnostico
sistema: central
tags: [whatsapp, openwa, gateway, erro]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 741
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.12 Número não exclui / conflito de FK (grupos e participantes)

**Sintoma.** A exclusão do número em `/whatsapp/numeros` falha em duas etapas: primeiro vem **`409 NUMERO_COM_HISTORICO`**; ao forçar, o `DELETE /api/v1/whatsapp/numeros/{id}` responde **`500`** por violação de FK e **o número continua na lista** — o histórico já foi apagado, mas a linha de `tbwhatsapp_numero` permanece.

**Causa.** As FKs `tbwhatsapp_grupo.WAT_NumeroId → tbwhatsapp_numero` e `tbwhatsapp_participante.WAT_GrupoId → tbwhatsapp_grupo` são `NO ACTION`. O `RemoverAsync` apagava eventos, mensagens, etiquetas, atendimentos e conversas, mas **não** os grupos, os participantes nem a fila da sessão — então o `DELETE tbwhatsapp_numero` era recusado pelo SQL Server e nada saía.

**Correção.** `WhatsAppNumeroService.RemoverAsync` passou a remover, na mesma `SaveChanges`:

- `tbwhatsapp_grupo` do número e `tbwhatsapp_participante` dos grupos desse número;
- `tbwhatsapp_webhook_fila` com o `SessaoId` do número.

Grupos, participantes e fila são **dados derivados**: saem **sempre**, mesmo sem `forcar`. O `409 NUMERO_COM_HISTORICO` continua valendo **só** para conversas, atendimentos e mensagens quando `forcar=false`; a contagem de mensagens agora também considera as ligadas por `AtendimentoId`.

**Fluxo da tela.** A exclusão abre um **modal próprio** (não usa `confirm()`): a primeira confirmação chama `removerNumero(id, false)`. Número **sem** histórico é removido ali mesmo; com histórico, o `409 NUMERO_COM_HISTORICO` aparece **no modal** com a mensagem de contagens e habilita a segunda confirmação **"Excluir com histórico"**, que chama `removerNumero(id, true)`. O endpoint continua aceitando `forcar=false` para API e outros consumidores. O gateway segue sendo removido **antes do `SaveChanges`** (`DELETE /api/sessions/{id}`, com queda para `POST .../stop`); se o gateway falhar, a exclusão no banco **prossegue** e a falha fica registrada no log.

**Testes.** `dotnet test --filter FullyQualifiedName~WhatsAppNumeroServiceTests` → **8 aprovados** (6 existentes + 2 novos: cascata com grupo/participante/fila e número que só tem grupo, sem conversas).

**Script e skill de limpeza.** Para zerar **todo** o módulo (e não um número), use `scripts/whatsapp-reset.ps1` — dry-run por padrão, `-Confirmar` executa, `-Ambiente AMBOS|HML|PROD` — e a skill `limpeza-whatsapp`. O roteiro, os parâmetros e o escopo por ambiente estão no **§19.11**; o gateway é **único** para HML e PROD, então apagar as sessões afeta os dois.

> ⚠️ Correção presente no working tree e **ainda sem deploy** nesta revisão: código pendente de validação final e commit.

