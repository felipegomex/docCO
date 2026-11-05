---
titulo: "Glossário central"
status: "canônico"
data: "2026-10-29"
---

# Glossário central

Termos que aparecem repetidos nos documentos canônicos. Cada termo traz definição curta, módulo relacionado e link para o canônico de origem. Termo sem fonte verificável fica em "Pendências", não nesta lista.

## Função

Categoria de trabalho do operador, **eixo do Kanban Geral** (`tbfuncao`). Cinco linhas na base real (2026-10-21): 1 `Analista de Sistemas`/`Implantador`, 2 `Suporte`/`Atendimento`, 3 `Programador`/`Desenvolvimento`, 4 `Executivo(a) Comercial` e 5 `CIAA`. É fonte do recorte do quadro e da permissão de edição; **não** é fonte de super-usuário (isso vem de `PERFIL_ID`). Substituiu o eixo Setor — ver [001-setor-para-funcao](./decisoes/001-setor-para-funcao.md).

- Módulo: Kanban Geral
- Fonte: `docs/arquitetura/banco-dados.md` — seções "Fontes de verdade" e "Contagens e fatos confirmados no baseline"

## Bloqueio

Atributo do card gravado em `tbbloqueio` (`BLQ_OrigemTipo` 0 Tarefa/1 Projeto/2 Etapa, `BLQ_OrigemId`, `BLQ_Motivo`, `BLQ_ResolvidoEm`/`BLQ_ResolvidoPor`). Índice único filtrado garante 1 bloqueio aberto por origem. Reabrir um item já bloqueado **atualiza** o motivo da linha aberta (histórico só existe entre ciclos fechar → abrir).

- Módulo: Kanban Geral
- Fonte: `docs/arquitetura/banco-dados.md` — seções "Inventário de entidades e tabelas" (`Bloqueios`) e migrations; `docs/operacao/troubleshooting.md` — seção 6, tabela de sintomas (motivo do bloqueio ao reabrir)

## Setor (histórico)

Eixo antigo do Kanban Geral, **removido**: a conversão Setor → Função (`20260929221556_SetorParaFuncao`) descartou `tbsetor`, `tbetapsetor` e as colunas `*_SetorId` e passou o quadro a usar `tbfuncao`. Mantido aqui para leitura de documentos e migrations antigas.

- Módulo: Kanban Geral
- Fonte: `docs/arquitetura/banco-dados.md` — seção "Migrations e tabelas históricas" (`20260929221556_SetorParaFuncao`); decisão em: [001-setor-para-funcao](./decisoes/001-setor-para-funcao.md)

## Prefixos de coluna do módulo Acessos (CAO_/RDP_/VPN_/ADK_/BCO_/AWB_)

Prefixo curto por tabela + `_` usado nas colunas das 6 tabelas do módulo Acessos Empresas: `CAO_` (`tbclienteacesso`), `RDP_` (`tbacessordp`), `VPN_` (`tbacessovpn`), `ADK_` (`tbacessoanydesk`), `BCO_` (`tbacessobanco`), `AWB_` (`tbacessoactyonweb`). Nomes de **tabela** não levam prefixo de módulo (só o `tb` geral).

- Módulo: Acessos Empresas
- Fonte: `docs/arquitetura/banco-dados.md` — seção "Módulo Acessos Empresas — nomenclatura, chaves e armadilhas" (tabela de prefixos)

## Cifrador

Interface `ICifrador` (`Cifrar`, `Decifrar`, `Implementacao`, `EhCifradoDeVerdade`) com duas implementações: `CifradorAesGcm` (**ativa** no DI) e `CifradorEmBranco` (**fallback de teste, não registrada**). `Decifrar` aceita linhas antigas sem o prefixo `v1:`; a cobertura do backfill precisa ser contada. Estado visível em `GET /api/v1/acessos/estado-cifra`. Decisão atual em: [005-cifra-aes-ativa](./decisoes/005-cifra-aes-ativa.md).

- Módulo: Acessos Empresas
- Fonte: `backend/Central_BackEnd/Services/Seguros/Cifrador.cs` (`ICifrador`); `docs/arquitetura/banco-dados.md` — seção "Cifra das senhas — `ICifrador` e como religar"

## Unidade

Registro de **nível 2** do módulo Acessos Empresas (`tbclienteacesso` / `ClienteAcesso`): representa a filial/unidade de um cliente do ERP. Aponta para `tbcliente` por `CAO_ClienteId` (índice único filtrado) e reúne os 5 tipos de acesso (RDP, VPN, AnyDesk, Banco e Actyon Web). A identidade (fantasia/CNPJ/grupo) não é gravada na unidade: vem da `tbcliente`.

- Módulo: Acessos Empresas
- Fonte: `backend/Central_BackEnd/Models/Acessos/ClienteAcesso.cs`; `docs/modulos/acessos.md` § 5

## Gate de senha

Fluxo que exige a senha do operador antes de operações sensíveis do Acessos Empresas. A prova é um ticket (`X-Acesso-Ticket`) emitido por `POST /clientes/{id}/autorizar`; `VISUALIZAR` vale 2 minutos (5 com “lembrar”) e `EDITAR` é de uso único. O usuário digitado é conferido contra o JWT e divergência devolve 401 sem queimar tentativa.

- Módulo: Acessos Empresas
- Fonte: `backend/Central_BackEnd/Services/Acessos/AcessoTicketService.cs`; `backend/Central_BackEnd/Services/PasswordValidationService.cs`; `docs/modulos/acessos.md` § 7

## Ticket de acesso

Prova de senha emitida por `AcessoTicketService` via `POST /clientes/{id}/autorizar`. `VISUALIZAR` vale 2 minutos (300 segundos quando `lembrar: true`) e é reutilizável na mesma empresa; `EDITAR` é de **uso único**, escopado por `ClienteAcessoId` **e** por `OperadorId` — a senha de uma empresa não abre outra.

- Módulo: Acessos Empresas
- Fonte: `docs/referencia/regras-negocio.md` — seção de gate de senha (`AcessoTicketService`); `docs/operacao/troubleshooting.md` — seção 10.4 (tabela de gate)

## Soft delete

Inativação por coluna `Ativo` (ou derivado, ex. `CAO_Ativo`), nunca remoção física. No Acessos: `DELETE` inativa e grava `UsuarioAlteracao`/`DataAlteracao`; a exclusão do cliente é **em cascata** (inativa o cliente e os 5 tipos de acesso). Nas FKs do módulo, `ON DELETE Restrict` é deliberado — "quem desativa é a coluna `Ativo`", nada é apagado em cascata.

- Módulo: Acessos Empresas
- Fonte: `docs/referencia/regras-negocio.md` — seção "Regras transversais (aplicadas em toda operação, em `AcessosService`)", itens 1–2; `docs/arquitetura/banco-dados.md` — seção do módulo Acessos (chaves e exclusões)

## Paginado<T>

Envelope de paginação (`items`, `totalCount`, `pageNumber`, `pageSize`, `totalPages`, `hasNext`, `hasPrev`) com `PaginadoDefaults.Normalizar` (default 20, máximo 200). Introduzido em `GET /api/v1/acessos/clientes`, o primeiro padrão de paginação do projeto — antes, todo endpoint devolvia `List<T>` completo.

- Módulo: Acessos Empresas (também usado pela auditoria)
- Fonte: `docs/referencia/regras-negocio.md` — seção "Paginação"; `docs/arquitetura/banco-dados.md` — seção "Refresh tokens e auditoria" (`tbauditoriaimplantacao`, paginado via `Paginado<T>`)

## Perfil

Papel de apresentação lido de `PERFIL_ID` em `tboperador` (`A`/`S`/`F`/`P`); o rótulo `F` no token passou a `Financeiro` e o `hasRole` do frontend compara também a letra crua. **Não é mais a fonte da permissão por função**: o Kanban usa o cargo `GESTOR_MASTER` + `tbgestor`; `SE_ADMIN`/`PERFIL_ID = 'A'` sobrevivem apenas como ponte para o Master. A **Função** (`FUNCAO_ID`) é o eixo do quadro, não a permissão.

- Módulo: transversal (autenticação e Kanban Geral)
- Fonte: `docs/referencia/regras-negocio.md` — seções "RBAC" e "Kanban Geral" (`FuncaoAuthorizationService`)

## Pendências

Nenhum termo pendente: os 10 termos do escopo da Fase 2 têm fonte verificada nos canônicos.
