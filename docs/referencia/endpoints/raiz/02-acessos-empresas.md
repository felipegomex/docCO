---
id: raiz.02
titulo: "Acessos Empresas"
tipo: referencia
area: api
sistema: central
tags: [api, endpoints, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests/Api
tokens_estimados: 2020
ficha: docs/modulos/acessos.md
---
> Parte de [Endpoints — Controllers na raiz](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

## Acessos Empresas

Fonte: [AcessosController.cs](../../../../backend/Central_BackEnd/Controllers/AcessosController.cs) (`IAcessosService` em [IAcessosService.cs](../../../../backend/Central_BackEnd/Services/Acessos/IAcessosService.cs), DTOs em [AcessosDtos.cs](../../../../backend/Central_BackEnd/Dtos/Acessos/AcessosDtos.cs)).

**31 operações** em `api/v{version:apiVersion}/acessos`, todas com `[Authorize]` na classe e **sem policy por papel**: qualquer operador autenticado cria, altera e inativa. A restrição é por auditoria, não por role. O CRUD de grupos saiu do módulo: a fonte de grupos passou a ser a legada `tbcliente_grupo`, somente leitura. Contagem `[Http*]` conferida em 2026-10-18 no código: 12 GET, 7 POST, 6 PUT, 6 DELETE.

### Gate de senha e diagnóstico

| Método | Rota | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|
| `POST` | `/clientes/{id}/autorizar` | `validacao` | `AutorizarRequest`: `usuario`, `senha`, `finalidade` (`VISUALIZAR` ou `EDITAR`), `lembrar` (bool, só em `VISUALIZAR`). O `usuario` é digitado no modal e **conferido contra o JWT** — divergência responde 401 sem queimar tentativa de brute force | `200 AutorizarResponse` (`ticket`, `expiraEm`, `finalidade`, `usoUnico`, `validoPor` em segundos); `400` finalidade inválida ou campos ausentes, `401` usuário/senha incorretos, `404` empresa inexistente, `429` em bloqueio por tentativas. É a **única porta de entrada** do módulo. TTL de `VISUALIZAR`: 120 s, ou **300 s com `lembrar: true`**; `EDITAR` continua uso único. `id = 0` emite ticket de criação, que só vale para `POST /clientes`. |
| `GET` | `/estado-cifra` | `leitura` | Sem body | `200 EstadoCifraResponse`: `implementacao`, `ehCifradoDeVerdade`, `eventosAuditoriaPerdidos`. Endpoint de diagnóstico da tela; é ele que permite avisar que as senhas estão em texto puro. |

O ticket devolvido viaja no header `X-Acesso-Ticket` nas operações sensíveis e é conferido por `RequerTicketAcessoAttribute`: `VISUALIZAR` dura 2 minutos e é reusável na mesma empresa; `EDITAR` dura 2 minutos e é de **uso único**, consumido na primeira operação aceita (logo, um segundo PUT legítimo na mesma empresa exige nova senha). O ticket é escopado por `ClienteAcessoId` e por `OperadorId`, de modo que a senha de uma empresa não abre outra.

### Grupos (`/grupos`)

Somente leitura — a fonte é a legada `tbcliente_grupo` (`GRUPO_ID smallint`). O módulo não cria, não edita e não remove grupos.

| Método | Rota | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/grupos` | `leitura` | Sem body | `200 List<GrupoResumo>` (`id`, `nome`), ordenado por nome |

### Clientes (`/clientes`)

| Método | Rota | Limite | Entrada | Resposta resumida |
|---|---|---|---|---|
| `GET` | `/clientes` | `leitura` | Query `grupoId?`, `busca?`, `somenteAtivos?`, `page?`, `pageSize?` | `200 Paginado<ClienteLista>`. **Primeiro endpoint paginado do projeto**: default 20, máximo 200. |
| `GET` | `/clientes/busca-erp` | `leitura` | Query `busca?`, `take?` | `200 List<ClienteErpResumo>` — typeahead da legada `tbcliente` (só ativos) por fantasia, razão social ou CNPJ. Termo com menos de 2 caracteres devolve lista vazia; `take` com clamp 1–50 (default 20). **Rota literal registrada antes de `clientes/{id:int}`.** |
| `GET` | `/clientes/{id}` | `leitura` | Path | `200 ClienteDetalhe` com as 5 listas de acesso e o resumo por tipo; `404` |
| `GET` | `/clientes/{id}/credenciais-completas` | `validacao` | Path | Operação presente no código (2026-10-18) sem linha no catálogo anterior — descrição de resposta a completar na próxima revisão, direto do código. |
| `POST` | `/clientes` | `validacao` | `ClienteCriarRequest`: **`clienteId` obrigatório** (empresa da `tbcliente` — não há cadastro manual de unidade), `observacao?` e **`acessos?`** (`AcessosAgrupadosRequest` com listas `rdp`, `vpn`, `anyDesk`, `banco`, `ActyonWeb`) | `201 ClienteCriarResponse` quando criou; **`200` com `jaExistia: true`** quando a empresa **já tinha** unidade (chave = `CLIENTE_ID`; nunca duplica). `400` se `clienteId` ausente, empresa inexistente na `tbcliente`, empresa inativa no ERP ou item de acesso fora do padrão (servidor/RDP min. 3, usuário min. 3, senha min. 6, porta 1–65535). Identidade (fantasia, razão social, CNPJ, grupo) **não vem no corpo nem é copiada**: a tela sempre lê da `tbcliente` por JOIN. **`acessos` preenchido ⇒ transação única**: cria a unidade (se preciso), todos os acessos, recalcula as flags `CAO_Tem*`, invalida o cache e audita um evento por registro — a operação inteira nasce ou morre junto. |
| `PUT` | `/clientes/{id}` | `validacao` | `ClienteAtualizarRequest`: **`ativo?`** e **`observacao?`** — só isso é editável | `204`; `400`/`404` (e `403` sem ticket `EDITAR`). Identidade (fantasia, CNPJ, grupo) pertence à `tbcliente` e é **ignorada** se vier no corpo: o operador corrige no ERP. Auditoria grava antes/depois com a identidade resolvida por JOIN. |
| `DELETE` | `/clientes/{id}` | `validacao` | Path | `204` — **soft delete em cascata**: inativa o cliente e os 5 tipos de acesso; `404` |

### Tipos de acesso

Cada tipo tem listar-por-cliente, criar, alterar, inativar. O corpo muda por tipo (`*_CriarRequest` / `*_AtualizarRequest`) e é validado no serviço.

| Tipo | Rotas | Observações |
|---|---|---|
| RDP | `GET /clientes/{clienteId}/rdp` · `POST /rdp` · `PUT /rdp/{id}` · `DELETE /rdp/{id}` | Exige **IP ou Servidor**; IPv4 validado; porta 1–65535 (default 3389); `usuario` e `senha` obrigatórios na criação |
| VPN | `GET /clientes/{clienteId}/vpn` · `POST /vpn` · `PUT /vpn/{id}` · `DELETE /vpn/{id}` | `tipoVpn` em `FortClient`/`OpenVPN`/`WireGuard`; `nome`, `usuario`, `vpnGateway` obrigatórios; senha opcional |
| AnyDesk | `GET /clientes/{clienteId}/anydesk` · `POST /anydesk` · `PUT /anydesk/{id}` · `DELETE /anydesk/{id}` | Nenhum campo obrigatório |
| Banco | `GET /clientes/{clienteId}/banco` · `POST /banco` · `PUT /banco/{id}` · `DELETE /banco/{id}` | `nomeBanco` e `servidor` obrigatórios; porta 1–65535 (default 1433) |
| Actyon Web | `GET /clientes/{clienteId}/actyon-web` · `POST /actyon-web` · `PUT /actyon-web/{id}` · `DELETE /actyon-web/{id}` | `url` obrigatória; porta 1–65535 (default 1010); `ambiente` em `PROD`/`HML`/`DEV` |

Todas as escritas devolvem `400` em falha de validação (`ArgumentException`) e `404` quando o registro não existe. Todas enfileiram auditoria.

### Revelação de senha — 1 rota, tipos rdp e vpn

| Método | Rota | Limite | Resposta |
|---|---|---|---|
| `GET` | `/clientes/{clienteId}/{tipo}/{acessoId}/senha` (`tipo` = `rdp` ou `vpn`) | `validacao` | `200 RevelacaoSenhaResponse` (`senha`, `tipo`, `usuario`, `clienteAcessoId`, `acessoId`, `aviso`) ou `404` |

Fato do código (2026-10-18): as duas rotas antigas (`GET /rdp/{id}/senha` e `GET /vpn/{id}/senha`) foram unificadas em **uma** rota com `tipo` no caminho — a operação é única na contagem `[Http*]`, e os contratos de resposta por tipo continuam os dois do quadro original do catálogo.

⚠️ **É o único endpoint do módulo que devolve credencial** (nos tipos `rdp` e `vpn`). Qualquer outro `GET` do módulo expõe apenas `temSenha: boolean`. Cada chamada enfileira `REVEAL_SENHA` **antes** de responder — mesmo quando o acesso não tem senha, a tentativa fica registrada. A resposta não traz a senha em nenhum outro campo.

**AnyDesk e Banco não têm endpoint de revelação**, apesar de gravar `*_SenhaCifrada`. É decisão deliberada: RDP e VPN são os tipos em que o suporte precisa consultar a senha guardada; AnyDesk e Banco são consultados por outros meios. Se for preciso revelar nesses tipos, é mudança de contrato, não correção de bug.

`DELETE` em qualquer tipo é soft delete (`Ativo = false`), nunca remoção física, e dispara `RecalcularFlagsAsync` no cliente.

