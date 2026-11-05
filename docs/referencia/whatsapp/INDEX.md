---
id: whatsapp.referencia
titulo: "Atendimento WhatsApp — implementação, contratos e operação"
tipo: referencia
area: atendimento
sistema: central
tags: [whatsapp, atendimento, servicos, contratos, testes, operacao]
status: ativo
revisado_em: "2026-11-03"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - frontend/src/app/features/whatsapp
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 366
ficha: docs/modulos/whatsapp.md
---

# Atendimento WhatsApp — implementação, contratos e operação

> Índice da referência profunda do módulo. Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).
> A norma de negócio está em [Regras de negócio — 16](../regras-negocio/16-modulo-de-atendimento-whatsapp-atribuicao-ciclo-de-vida-e-ag.md) e [25](../regras-negocio/25-atendimento-whatsapp-protocolo-foto-telefone-e-schema.md); o contrato HTTP, em [Endpoints WhatsApp](../endpoints/whatsapp/INDEX.md), [17](../endpoints/whatsapp/17-observacoes-de-contrato.md) e [18](../endpoints/whatsapp/18-observacoes-de-contrato-gateway-e-rodadas.md). Este documento guarda o detalhe de implementação, o mapeamento de edição, a evidência de teste e o histórico operacional que antes sobrecarregavam a ficha.

## Partes

- [Serviços, componentes e mapeamento de edição](./01-servicos-e-mapeamento.md)
- [Regras de implementação e contratos](./02-regras-e-contratos-do-modulo.md)
- [Testes, pendências e execução operacional](./03-testes-pendencias-e-operacao.md)
