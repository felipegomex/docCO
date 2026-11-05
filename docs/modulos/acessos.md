---
id: mod.acessos
titulo: "Acessos Empresas"
tipo: modulo
area: acessos
sistema: central
tags: [acessos, cliente, unidade, senha, gate, cifra, auditoria, rdp, vpn, anydesk, banco, actyon]
dono: suporte
status: ativo
revisado_em: "2026-10-29"
fonte: [backend/Central_BackEnd/Controllers/AcessosController.cs, backend/Central_BackEnd/Services/Acessos, backend/Central_BackEnd/Models/Acessos, frontend/src/app/features/acessos]
testes: [backend/Central_BackEnd.Tests/AcessosUnidadeTests.cs, backend/Central_BackEnd.Tests/TicketAcessoTests.cs, backend/Central_BackEnd.Tests/TemSenhaTests.cs]
tokens_estimados: 1900
---

# Acessos Empresas

## 1. Quando usar este card

Unidade/filial de cliente, credenciais RDP/VPN/AnyDesk/Banco/Actyon Web, gate de
senha, ticket `X-Acesso-Ticket`, cifra, revelação de credencial e auditoria.

## 2. Propósito e escopo

Cadastra a **unidade** (nível 2) e seus **5 tipos de acesso** (nível 3), com gate
de senha, cifra e auditoria. Não cobre o cadastro da empresa no ERP (`tbcliente`,
somente leitura) nem o CRUD de grupos (`tbcliente_grupo`, legado somente leitura).

## 3. Rotas do frontend

| URL | Componente | Fonte |
|---|---|---|
| `/acessos` | redirect para `/acessos/clientes` | `frontend/src/app/features/acessos/acessos.routes.ts` |
| `/acessos/clientes` | `ListaClientesCardsComponent` (padrão) | idem |
| `/acessos/clientes/tabela` | `ListaClientesComponent` | idem |
| `/acessos/clientes/:id` | `DetalheClienteComponent` | idem |

Criação via `FormClienteModalComponent`.

## 4. Endpoints do backend

`AcessosController` em `/api/v{version}/acessos`, **31 operações**, `[Authorize]`
na classe e sem policy de papel.

| Grupo | Rotas | Autorização | Fonte |
|---|---|---|---|
| Gate | `POST /clientes/{id}/autorizar`; `GET /estado-cifra` | `[Authorize]` | `backend/Central_BackEnd/Controllers/AcessosController.cs` |
| Grupos | `GET /grupos` | `[Authorize]` | idem |
| Clientes | `GET /clientes`, `/clientes/busca-erp`, `/clientes/{id}`, `/credenciais-completas`, `POST/PUT/DELETE` | ticket nas leituras/escritas sensíveis | idem |
| Tipos | `GET /clientes/{clienteId}/{tipo}`, `POST /{tipo}`, `PUT/DELETE /{tipo}/{id}` (rdp, vpn, anydesk, banco, actyon-web) | `[Authorize]` + `RequerTicketAcesso` | idem |
| Revelação | `GET /clientes/{clienteId}/{tipo}/{acessoId}/senha` | ticket de visualização | idem |

## 5. Tabelas e entidades

| Classe → tabela | Papel | Fonte |
|---|---|---|
| `ClienteAcesso` → `tbclienteacesso` | Unidade (nível 2) | `backend/Central_BackEnd/Models/Acessos/ClienteAcesso.cs` |
| `AcessoRdp` → `tbacessordp`; `AcessoVpn` → `tbacessovpn` | RDP e VPN | `backend/Central_BackEnd/Models/Acessos/Acessos.cs` |
| `AcessoAnyDesk` → `tbacessoanydesk`; `AcessoBanco` → `tbacessobanco` | AnyDesk e Banco | idem |
| `AcessoActyonWeb` → `tbacessoactyonweb` | Actyon Web (sem senha); `AcessoBase` é a base abstrata | idem |
| `AuditoriaAcesso` → `tbauditoriaacesso` | Trilha de revelação | `backend/Central_BackEnd/Models/Acessos/AuditoriaAcesso.cs` |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | `AcessosService` (+ partial `AcessosConcretos`) | Clientes, tipos, flags e revelação | `backend/Central_BackEnd/Services/Acessos/AcessosService.cs` |
| backend | `AcessoTicketService` | Emite/valida o ticket | `backend/Central_BackEnd/Services/Acessos/AcessoTicketService.cs` |
| backend | `PasswordValidationService` | Valida senha contra `tboperador` | `backend/Central_BackEnd/Services/PasswordValidationService.cs` |
| backend | `CifradorAesGcm` / `ICifrador` | Cifra `*_SenhaCifrada` | `backend/Central_BackEnd/Services/Seguros/Cifrador.cs` |
| backend | `RequerTicketAcessoAttribute` | Exige ticket | `backend/Central_BackEnd/Authorization/RequerTicketAcessoAttribute.cs` |
| frontend | `AcessosService` | HTTP do módulo | `frontend/src/app/features/acessos/acessos.service.ts` |
| frontend | `SenhaGateService` | Pede senha e emite ticket | `frontend/src/app/core/services/senha-gate.service.ts` |
| frontend | `ModalSenhaComponent` | Modal de autorização | `frontend/src/app/features/acessos/components/modal-senha/modal-senha.component.ts` |

## 7. Regras de negócio

- Unidade nasce do ERP: `clienteId` obrigatório; não há cadastro manual
  (`AcessosService#CriarClienteAsync`).
- No máximo uma unidade por empresa (índice único filtrado em `CAO_ClienteId`);
  repetição devolve `jaExistia` e abre a existente.
- Identidade vem da `tbcliente`; flags `CAO_Tem*` são derivadas e recalculadas a
  cada mutação.
- `DELETE` é soft delete em cascata; inativa, não apaga.
- Gate: `VISUALIZAR` 2 min reutilizável (5 min com “lembrar”), `EDITAR` 2 min de
  uso único (`AcessoTicketService`: `TtlVisualizar`, `TtlVisualizarLembrar`,
  `TtlEditar`).
- Usuário digitado é conferido contra o JWT; divergência é 401 sem queimar
  tentativa.
- Só RDP e VPN revelam senha; a revelação enfileira `REVEAL_SENHA`.

## 8. Autorização

- `[Authorize]` de classe, **sem policy por papel**; a segregação é por auditoria.
- Operações sensíveis exigem `X-Acesso-Ticket`; sem ticket, 403.
- `GET /api/v1/auditoria/acessos` exige admin ou gestor de função; `/auditoria`
  não tem guard de papel de propósito.

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| “criar unidade” | `backend/Central_BackEnd/Services/Acessos/AcessosService.cs#CriarClienteAsync` | `frontend/src/app/features/acessos/pages/form-cliente-modal/form-cliente-modal.component.ts#FormClienteModalComponent` | `backend/Central_BackEnd.Tests/AcessosUnidadeTests.cs` |
| “adicionar tipo de acesso” | `backend/Central_BackEnd/Services/Acessos/AcessosService.AcessosConcretos.cs#CriarRdpAsync` | `frontend/src/app/features/acessos/pages/form-acesso/form-acesso.component.ts#FormAcessoComponent` | `backend/Central_BackEnd.Tests/AcessosCrudCaracterizacaoTests.cs` |
| “ticket/TTL” | `backend/Central_BackEnd/Services/Acessos/AcessoTicketService.cs#Emitir` | `frontend/src/app/core/services/senha-gate.service.ts#SenhaGateService` | `backend/Central_BackEnd.Tests/TicketAcessoTests.cs` |
| “gate de senha” | `backend/Central_BackEnd/Services/PasswordValidationService.cs#Validar` | `frontend/src/app/features/acessos/pages/modal-senha/modal-senha.component.ts#ModalSenhaComponent` | `backend/Central_BackEnd.Tests/ValidadoresAcessoTests.cs` |
| “revelar credencial” | `backend/Central_BackEnd/Services/Acessos/AcessosService.cs#RevelarSenhaAsync` | `frontend/src/app/features/acessos/pages/visualizar-acesso-modal/visualizar-acesso-modal.component.ts#VisualizarAcessoModalComponent` | `backend/Central_BackEnd.Tests/AcessosCrudCaracterizacaoTests.cs` |
| “flags CAO_Tem*” | `backend/Central_BackEnd/Services/Acessos/AcessosService.cs#RecalcularFlags` | — | `backend/Central_BackEnd.Tests/AcessosUnidadeTests.cs` |

## 10. Contratos que não podem quebrar

- `POST /clientes` com `acessos` é **uma transação**: unidade + acessos + flags +
  cache + auditoria, ou nada.
- `SenhaCifrada`/`TemSenha` ficam em cada classe concreta, **nunca** em
  `AcessoBase` (mover quebra `TemSenha`).
- `tbacessoactyonweb` não tem `SenhaCifrada`.
- A senha só sai pelos endpoints de revelação; os demais `GET` expõem
  `temSenha: boolean`.
- O ticket é escopado por unidade + operador; auditoria nunca grava a senha.

## 11. Armadilhas conhecidas

- `dotnet ef migrations add` não deve ser rodado: snapshot do EF divergente.
- `CAO_Cnpj`/`CAO_Fantasia` foram dropadas; são 31.
## 12. Testes

- `AcessosUnidadeTests`, `AcessosCrudCaracterizacaoTests`, `ValidadoresAcessoTests`,
  `TicketAcessoTests` e `TemSenhaTests`.
- QA: `docs/referencia/qa/14-9-acessos-empresas.md` (CF-04 e CF-12).

## 13. Orçamento de leitura

Abra esta ficha; depois só o arquivo da seção 9 e o teste. Limite: 1 ficha,
4 trechos de código e ≤ 12 tool calls.

## 14. Pendências

- Termos confirmados no glossário; modal com spec de UI.
