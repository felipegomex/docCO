# Evolução WhatsApp — Pendências para implantação

**Branch:** developer
**Gateway:** OpenWA v0.25.0
**Atualizado em:** 03/11/2026
**Status:** pendente de implantação (nada implementado ainda)

Este documento lista apenas o que falta implantar. Itens já concluídos ou descartados foram removidos.

---

## 0. Pré-requisitos

- Há alterações não commitadas em arquivos WhatsApp (`AtendimentoService.cs`, `AtendimentoService.CicloVida.cs`, `WhatsAppAtendimentosController.cs`, `WhatsAppNumeroService.cs`, testes e `WhatsApp_Reset_Dados_Idempotente.sql`). Fechar ou commitar antes de começar, para não misturar escopo.
- Os números de linha antigos do plano anterior podem ter mudado. Conferir símbolos, não linhas.

## 1. Decisões pendentes (responder antes de implementar)

| # | Pergunta | Afeta |
|---|---|---|
| D1 | A leitura de mensagem por operador pode virar uma tabela nova (`tbwhatsapp_leitura_mensagem_operador` ou nome equivalente)? Exige migration e pedido explícito. Alternativa sem banco: não há. | Prompt 1 |
| D2 | `historico-cliente` existente atende o Prompt 6, ou criamos `WhatsAppHistoricoController` novo? | Prompt 6 |
| D3 | "Tentar novamente" entra no escopo do Prompt 2? O reenvio já existe em `EnviarPelaCentralAsync`. | Prompt 2 |

## 2. Prompt 2 — Status de envio das mensagens

### 2.1 Já existe (reutilizar)
- `OpenWaClient.EnviarTextoAsync` (`backend/Central_BackEnd/Services/WhatsApp/OpenWaClient.cs`): envia `{ chatId, text }` e retorna `OpenWaEnvioDto` (`Dtos/WhatsApp/OpenWaDtos.cs`).
- `AtendimentoService`: grava `Mensagem.Status` (`StatusMensagemWhatsApp`: Pendente, Enviada, Falhou, etc.) e `Mensagem.ExternalId`.
- `WhatsAppWebhookService`: trata `message.ack` (`OpenWaAckPayload`) com avanço monotônico (`DeveAtualizarStatus`) e exceção controlada para `Falhou`.
- Frontend: `whatsapp-message.component.ts` já reconhece `MENSAGEM_FALHOU` (classe `wa-balao__entrega--falhou`).

### 2.2 Limites confirmados do gateway
- `quotedMessageId` no envio **não existe** no OpenWA 0.25.0. Reply/citação está descartado.
- `send-contact` não foi testado na prática (fallback vCard para 404/501).
- Idempotência (`X-OpenWA-Idempotency-Key`) é enviada, mas a capacidade no gateway não foi confirmada (pendência P4).

### 2.3 A desenvolver
**Backend**
- Expor `status` e `enviadaEm` na resposta dos envios (`AtendimentoDetalhe`), se ainda não expostos.
- Validar na prática se o gateway envia `message.ack` para todos os tipos de mensagem.

**Frontend**
- `whatsapp-message.component`: indicador de status (enviando, enviada, entregue, lida, falha).
- `whatsapp-composer.component`: estado otimista "enviando" enquanto aguarda resposta.
- `whatsapp.service.ts` (`enviarMensagem`, linha ~215): garantir retorno de `status` e `enviadaEm`.
- Botão "Tentar novamente": somente para `Falhou` sem `ExternalId`. Depende de D3.

### 2.4 Regras
1. Avanço monotônico: evento atrasado não rebaixa status já conhecido.
2. `Falhou` só substitui Pendente/Enviada; entrega ou leitura posterior substitui a falha.
3. Entregue/lida somente com confirmação do webhook `message.ack`.
4. Erro ao cliente é genérico; detalhe técnico só no log.
5. "Tentar novamente" apenas quando seguro.

### 2.5 Critérios de aceite
1. Indicador visual ao lado de cada mensagem enviada pela Central.
2. Entregue/lida só quando o webhook confirmar.
3. Falha identificável, com erro genérico ao operador.
4. Notificações fora de ordem não rebaixam status.
5. "Tentar novamente" só quando seguro.

### 2.6 Riscos
- Presumir entrega/leitura para tipos de mensagem que o gateway não confirma.
- Evento atrasado sobrescrevendo status mais novo.

---

## 3. Prompt 1 — Não lidas e atendimentos pendentes

### 3.1 Já existe (reutilizar)
- `Atendimento.MensagensNaoLidas` (`WAT_MensagensNaoLidas`): contador **global por atendimento**, não por operador.
- `Atendimento.UltimaMensagemEm`, `UltimaMensagemTrecho`, `PrimeiraRespostaEm`, `ResponsavelDesde`, `Status`.
- `whatsapp.utils.ts` (~linha 306): lista já ordena conversas com não lidas primeiro.
- SignalR: `whatsapp-realtime.service.ts` e `WhatsAppRealtimeService.cs`.
- Dedupe por `ExternalId` no webhook.

### 3.2 A desenvolver
1. **Leitura por operador:** depende de D1. Endpoint para marcar como lida pelo operador logado (ex.: `PATCH /whatsapp/atendimentos/{id}/ler`). O backend valida o operador.
2. **Não lidas por operador:** o contador atual é global. Calcular ou persistir o contador individual conforme D1.
3. **Aguardando suporte x aguardando cliente:** reutilizar a regra existente (status `AguardandoCliente` e coluna do kanban). Não criar campo novo.
4. **Tempo de espera:** `PrimeiraRespostaEm` ou última mensagem do cliente. A mensagem `UltimaMensagemClienteEm` **não existe**; é preciso decidir se usa `UltimaMensagemEm` ou criar campo (exige migration).
5. **Frontend:** badge de não lidas por conversa, destaque visual e atualização via SignalR, sem recarregar a página.

### 3.3 Regras
1. Leitura individual por operador.
2. Nova mensagem recebida incrementa o contador de quem tem acesso ao atendimento.
3. Abrir a conversa e marcar como lida zera o contador só para quem leu.
4. `Finalizado` não aparece em pendentes.
5. `ExternalId` duplicado não incrementa.

### 3.4 Critérios de aceite
1. Nova mensagem atualiza o indicador da conversa.
2. Leitura atualiza a contagem conforme a regra individual.
3. Indicadores permanecem após atualizar a página.
4. Tempo de espera calculado a partir de referência confiável.
5. Aguardando cliente não aparece como aguardando suporte.
6. Eventos duplicados não aumentam a contagem.
7. Atualização automática via SignalR.
8. Respeita permissões dos operadores.

### 3.5 Riscos
- Estado inconsistente após recarregar se a leitura não for persistida.
- Concorrência entre operadores.
- Migration necessária (D1).

---

## 4. Prompt 6 — Histórico de atendimentos finalizados por empresa

### 4.1 Modelo confirmado
```
ClienteLegado (ERP, tabela de empresas)
  └─ Contato.EmpresaId (WAT_EmpresaId)
       └─ Conversa (WAT_ChatId)
            └─ Atendimento (Status = Finalizado)
                 └─ Mensagem
```
- `cliente_id` **não está** em `Atendimento`. O vínculo é `Contato.EmpresaId`, então a consulta precisa de JOIN.
- Entidade de empresa: `ClienteLegado` (`backend/Central_BackEnd/Models/Implantacao/ClienteLegado.cs`), não `ClientesLegado`.

### 4.2 Já existe (reutilizar)
- `GET /whatsapp/atendimentos/{id}/historico-cliente` (`whatsapp.service.ts` ~306): histórico de todos os ciclos do cliente. Verificar se atende o caso (D2).
- `GET /whatsapp/contatos/empresas?busca=&take=` (typeahead de empresas, `whatsapp.service.ts` ~706).
- `GET /whatsapp/grupos/{id}/historico` e `GET /whatsapp/admin/contatos/{id}/historico` (admin).

### 4.3 A desenvolver (se D2 indicar controller novo)
1. `GET /whatsapp/historico/empresas?busca=`: pesquisa de empresas.
2. `GET /whatsapp/historico/empresas/{empresaId}/atendimentos`: somente `Finalizado`, paginado, ordenado por data.
3. `GET /whatsapp/historico/atendimentos/{id}/mensagens`: leitura somente.
4. Filtro de `Finalizado` no backend, não só no frontend.
5. Frontend: pesquisa por empresa, listagem (data, contato, responsável, data de encerramento, prévia), visualização somente leitura sem envio, anexos, reações ou reabertura.

### 4.4 Regras
1. Somente `Finalizado`.
2. Todos podem consultar (sem restrição por empresa).
3. Somente leitura.
4. Mostrar contatos e números diferentes da mesma empresa.
5. Não misturar empresas.
6. Contato sem vínculo com empresa: documentar a limitação.
7. Atendimento reaberto segue a regra oficial de status.

### 4.5 Critérios de aceite
1. Pesquisar empresa e ver seus atendimentos finalizados.
2. Nenhum atendimento não finalizado aparece.
3. Diferentes contatos e números da mesma empresa aparecem juntos.
4. Empresas diferentes não se misturam.
5. Consulta sem risco de envio acidental.
6. Empresa sem finalizados e nomes semelhantes são tratados.
7. Permissão de consulta respeitada.
8. Paginação para empresas com muitos atendimentos.
9. Dados históricos preservados.

### 4.6 Riscos
- Vínculo empresa-contato inconsistente.
- Telefones duplicados.
- Atendimentos antigos sem vínculo confiável.

---

## 5. Prompt 4 — Pesquisa dentro da conversa

### 5.1 Já existe (reutilizar)
- `WhatsAppAtendimentosController`: `GET /atendimentos/{id}` carrega conversa.
- `Mensagem` (`backend/Central_BackEnd/Models/WhatsApp/Mensagem.cs`): `Texto`, `Tipo`, `AutorNome`, `EnviadaEm`, `Direcao`, `MidiaMime`. (O campo é `Texto`, não `Body`.)
- `whatsapp.service.ts` ~linha 417: já há busca de conversas por `busca`, como referência de padrão.

### 5.2 A desenvolver
1. Endpoint `GET /whatsapp/atendimentos/{id}/mensagens/busca?termo=&pagina=&tamanho=`, com paginação no backend.
2. Busca case-insensitive, sem acento e parcial. Confirmar se o banco permite isso sem regressão de desempenho.
3. Pesquisar `Texto` e legenda de mídia. Verificar onde a legenda é gravada.
4. Frontend: campo de busca na conversa, resultados com autor e data, navegação entre resultados, destaque do termo, localizar a mensagem na conversa, estado sem resultados.
5. Permissão de acesso ao atendimento revalidada no backend.

### 5.3 Riscos
- Conversas longas: paginação obrigatória.
- Mensagens antigas podem não estar carregadas no navegador.
- Mídia sem legenda não é pesquisável como texto.

---

## 6. Ordem sugerida

| Ordem | Item | Depende de |
|---|---|---|
| 1 | Prompt 2 — Status de envio | D3 (só para "Tentar novamente") |
| 2 | Prompt 1 — Não lidas | D1 |
| 3 | Prompt 6 — Histórico por empresa | D2 |
| 4 | Prompt 4 — Busca na conversa | Nenhum |

## 7. Fora do escopo por decisão

- Responder/citar mensagem (Prompt 3): sem suporte no envio do OpenWA 0.25.0.
- Presença dos operadores (Prompt 5): fica para o futuro.

## 8. Ao implantar

- Mudanças de schema exigem pedido explícito e migration idempotente.
- Validar com `scripts\validate.ps1`.
- Atualizar a documentação em `docs/` pelo `docs-writer`.
