---
id: 20-19-gateway-openwa-canal-de-whatsapp.16
titulo: "19.15 Foto do contato não aparece (Fase 3)"
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
tokens_estimados: 835
ficha: docs/modulos/whatsapp.md
---
> Parte de [19. Gateway OpenWA (canal de WhatsApp)](./INDEX.md). Ficha: [Atendimento WhatsApp](../../../modulos/whatsapp.md).

### 19.15 Foto do contato não aparece (Fase 3)

**Sintoma.** O avatar mostra as iniciais (ou o ícone de grupo) mesmo quando o contato tem foto no WhatsApp; `GET /api/v1/whatsapp/contatos/{id}/foto` responde **404**, e o log do backend registra falha ao obter/baixar a imagem.

**Causa provável (na ordem em que se confere).**

1. **Contato sem conversa.** O `WhatsAppFotoContatoService` resolve a sessão pela **conversa mais recente** (`Conversa.NumeroId` → `Numero.SessaoId`). Sem conversa (ou sem `SessaoId` no número), não há como perguntar ao gateway: se não existir uma `WAT_FotoUrl` antiga ainda baixável, a rota responde `404`.
2. **Gateway sem foto ou rota indisponível.** `ObterFotoPerfilAsync` (`GET /api/sessions/{sessao}/contacts/{chatId}/profile-picture`) devolve `null` em `404`, falha de rede, timeout ou resposta inesperada — o serviço grava o **cache negativo** em `WAT_FotoAtualizadaEm` e só vai tentar de novo depois do TTL `atendimento.fotoTtlHoras` (padrão **24 h**). Para forçar o teste, zere `WAT_FotoAtualizadaEm`/`WAT_FotoUrl` do contato (ou reduza o parâmetro) e abra a tela de novo.
3. **URL expirada.** A URL do gateway é temporária. Se o download falhar, o serviço faz **uma** nova consulta e tenta de novo; se a segunda falhar, responde `404` até o próximo ciclo.
4. **Imagem recusada.** O download exige `Content-Type` `image/*` (SVG recusado), tamanho máximo de **2 MB** e URL absoluta `http`/`https`. Content-type genérico, SVG ou arquivo grande são descartados (`null`) sem derrubar a tela; falha de rede/exceção é logada.
5. **Chave da API ausente/incorreta.** O download no servidor usa o mesmo `WhatsApp:ApiKey` (header `X-API-Key`). Se a chave estiver vazia, a URL do gateway ainda é tentada; se o gateway exigir a chave, o download falha. Confira `WhatsApp__ApiKey` — **nunca** registre o valor no chamado.
6. **Cache do navegador/tela.** O backend responde com `Cache-Control: private, max-age=600` e o frontend reaproveita o blob por contato (`shareReplay`). Um contato que falhou antes não fica em cache (erro é removido do `Map`), mas o teste pode exigir recarregar a tela; o backend ainda pode ter o cache em memória de **10 min** por contato.

**Diagnóstico.** Abra o Network e confira o status/content-type de `/api/v1/whatsapp/contatos/{id}/foto`; consulte `WAT_FotoUrl`, `WAT_FotoAtualizadaEm` e `WAT_TelefoneResolvido` do contato; confira o parâmetro `atendimento.fotoTtlHoras`; e procure no log o aviso de falha ao obter/baixar a foto. O endpoint é autenticado: `401` é sessão/token, `404` é “sem foto/indisponível”, e a URL do gateway **não** aparece na resposta ao navegador.

**Importante.** A foto **não** é pré-requisito de atendimento: sem ela, o avatar usa `iniciaisDoNome`/`corDoNome` e grupo usa `bi-people`. Para `@lid`, o telefone resolvido exibido (`WAT_TelefoneResolvido`) é **display** e não muda a chave de identidade. A Fase 3 **não** cria script novo: usa as colunas `WAT_FotoUrl`/`WAT_FotoAtualizadaEm` do `WhatsApp_Fase8_Idempotente.sql` (§19.13).

