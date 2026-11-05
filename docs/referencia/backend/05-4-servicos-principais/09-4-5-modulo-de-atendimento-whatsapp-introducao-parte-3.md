---
id: 05-4-servicos-principais.09
titulo: "4.5 Módulo de atendimento WhatsApp — introdução — parte 3"
tipo: referencia
area: backend
sistema: central
tags: [backend, services, di, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Configuration
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 258
ficha: docs/modulos/whatsapp.md
---
> Parte de [4. Serviços principais](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 4.5 Módulo de atendimento WhatsApp — introdução — parte 3

| Serviço | Responsabilidade | Dependências | Validações confirmadas | Efeitos colaterais confirmados |
|---|---|---|---|---|
| `WhatsAppAssistenteIaService` (`IWhatsAppAssistenteIaService`) | **Fase G**: `ObterAsync(atendimentoId)` — chama `GeminiAnaliseProvedor` e devolve resumo, classificação (categoria, prioridade, sentimento, intenção) e sugestões (respostas, artigos, procedimentos, próximos passos) | `AppDbContext`, `GeminiAnaliseProvedor` (DI), logger | Atendimento deve existir e ser acessível; provedor não configurado → `404` "em breve" | Integração com provedor existente; não persiste resultado; falha do provedor → `502`/`504` |

