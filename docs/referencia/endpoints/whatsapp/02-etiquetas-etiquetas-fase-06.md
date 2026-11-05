---
id: whatsapp.02
titulo: "Etiquetas (`/etiquetas`) — Fase 06"
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
tokens_estimados: 456
ficha: docs/modulos/whatsapp.md
---
> Parte de [Endpoints — Controllers de atendimento WhatsApp](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

## Etiquetas (`/etiquetas`) — Fase 06

Catálogo **novo e exclusivamente operacional** (não restaura a taxonomia removida na Fase D). A **leitura é de qualquer atendente** — quem aplica precisa ver as opções; **criar e editar são de administrador** (D06), revalidado no controller. A aplicação ao atendimento fica em `/atendimentos/{id}/etiquetas` (escrita no atendimento), não neste controller.

| Método | Rota | Autorização | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/etiquetas` | atendente | Query `somenteAtivas` (padrão `true`), `busca?` | `200 List<EtiquetaWhatsApp>` (`id`, `nome`, `cor`, `ativo`), ordenado por nome |
| `POST` | `/etiquetas` | **administrador** | Body `CriarEtiquetaRequest`: `nome` (≤ 40), `cor` | `201 EtiquetaWhatsApp`; `400` nome/cor inválidos; **`409` `ETIQUETA_JA_EXISTE`** (nome repetido entre ativas) |
| `PATCH` | `/etiquetas/{id}` | **administrador** | Body `AtualizarEtiquetaRequest`: `nome?`, `cor?`, `ativo?` (nulo mantém) | `200 EtiquetaWhatsApp`; `404`; `400` validação; `409` nome repetido. **Inativar não apaga** — os vínculos históricos continuam |

A `cor` é uma **lista fechada** do design system (`azul`, `amarelo`, `laranja`, `roxo`, `verde`, `vermelho`, `neutro`), não HTML livre; o banco tem `CHECK` com os mesmos valores. O nome é normalizado (trim + espaços colapsados) e único **entre as ativas**. Etiqueta inativa continua aparecendo no detalhe do atendimento quando já estava vinculada, mas não pode ser adicionada a outro.

