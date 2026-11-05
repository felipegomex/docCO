---
id: regras-negocio.21
titulo: "Módulo Acessos Empresas"
tipo: referencia
area: negocio
sistema: central
tags: [regras, negocio, permissoes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services
  - backend/Central_BackEnd/Controllers
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1759
ficha: docs/modulos/acessos.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Acessos Empresas](../../modulos/acessos.md).

## Módulo Acessos Empresas

**Situação: implementado; CRUD completo em 6 tabelas (unidade + 5 tipos), soft delete e auditoria em fila.**

Estrutura: **grupo econômico (legado `tbcliente_grupo`, somente leitura) → cliente/unidade (com `CAO_ClienteId` para a empresa do ERP em `tbcliente`) → 5 tipos de acesso** (RDP, VPN, AnyDesk, banco, Actyon Web). Antes lia uma planilha do Google Sheets com três endpoints somente leitura; hoje tudo é persistido no `AppDbContext` (a antiga `tbgrupoacesso` foi dropada e o módulo passou a criar unidade a partir de uma empresa do ERP). Detalhes de dados em [`04-ESTRUTURA-DADOS.md`](../../arquitetura/banco-dados.md) e de tela em [`qa-historias.md`](../qa-historias.md).

### Autorização

- `[Authorize]` na classe `AcessosController`, **sem policy por papel**: qualquer operador autenticado cria, altera e inativa. O projeto **tem** policies de autorização no RBAC (`perm:<recurso>:<acao>`, ver a seção própria), mas o módulo Acessos ainda não usa nenhuma.
- A restrição real é **por auditoria**, não por role: toda mutação enfileira `CREATE`/`UPDATE`/`DELETE` e toda revelação de senha enfileira `REVEAL_SENHA` com usuário, IP e user-agent.
- O gate de senha (`POST /clientes/{id}/autorizar`) é uma **segunda barreira de uso**, não de autorização: o corpo traz `usuario` digitado (conferido contra o JWT — divergência responde 401 auditada **sem** queimar tentativa de brute force) e a senha, comparada pelo `PasswordValidationService`, que **não guarda cache nenhum** e delega o bloqueio por tentativas ao `BruteForceGuard`. A prova de senha que viaja depois é o **ticket de acesso** (`AcessoTicketService`): `VISUALIZAR` vale 2 minutos (reutilizável na mesma empresa), **300 segundos quando `lembrar: true`** (`TtlVisualizarLembrar`), e `EDITAR` é de **uso único**, escopado por `ClienteAcessoId` **e** por `OperadorId`. Por isso a senha de uma empresa não abre outra, e cada escrita ou revelação consome o ticket dela.

### Regras transversais (aplicadas em toda operação, em `AcessosService`)

1. **Soft delete, nunca remoção física.** `DELETE` inativa (`Ativo = false`) e grava `UsuarioAlteracao`/`DataAlteracao`.
2. **Soft delete em cascata no cliente.** `DELETE /clientes/{id}` inativa o cliente **e os 5 tipos de acesso** dele, para não deixar credencial ativa apontando para unidade inativa.
3. **Flags `CAO_Tem*` são DERIVADAS.** `RecalcularFlagsAsync` roda após toda mutação de acesso e após editar o cliente; o consumidor nunca as envia. Se fossem editáveis, inativar o único RDP deixaria `CAO_TemRdp = true`.
4. **Senha só sai pelo endpoint de revelação.** Nos DTOs de leitura existe apenas `temSenha: bool`; nunca o valor.
5. **Revelação só existe em RDP e VPN** (`GET /rdp/{id}/senha` e `GET /vpn/{id}/senha`). AnyDesk e Banco têm `*_SenhaCifrada` gravada, mas **não** têm endpoint de revelação: a credencial existe e é auditável no ato de gravar, mas não é exibida. A restrição é deliberada — são os dois tipos em que o suporte realmente precisa consultar a senha guardada.
6. **Auditoria é assíncrona e não derruba a operação.** Falha ao gravar o lote só gera `LogError`; a mutação de negócio já foi confirmada.
7. **Grupos são somente leitura.** A fonte é a legada `tbcliente_grupo`; o módulo não cria, não edita e não remove grupo (o CRUD antigo de `tbgrupoacesso` foi removido junto com a tabela). `ValidarGrupoAsync` devolve `400` se o grupo informado não existir ou estiver inativo, e `POST /clientes` com `clienteId` herda o `GRUPO_ID` da empresa no ERP (fica `NULL` se ela não tem grupo).

### Validações por tipo

| Tipo | Regras |
|---|---|
| RDP | Exige **IP ou Servidor** (ambos vazios → `400`). IP validado como IPv4. Porta 1–65535, default **3389**. Usuário e senha são obrigatórios na criação. |
| VPN | `TipoVpn` validado contra `TiposVpn` = `FortClient`/`OpenVPN`/`WireGuard`; `Nome`, `Usuario` e `VpnGateway` obrigatórios. Senha **opcional**. |
| AnyDesk | Nenhum campo obrigatório; senha opcional. |
| Banco | `NomeBanco` e `Servidor` obrigatórios. Porta 1–65535, default **1433**. `VersaoSql`/`VersaoCob`/`ArquivoConfig` opcionais. |
| Actyon Web | `Url` obrigatória. Porta 1–65535, default **1010**. `Ambiente` validado contra `AmbientesAcesso` = `PROD`/`HML`/`DEV`. **Não tem usuário nem senha** — o acesso é por URL/sessão. |

`TipoVpn`, `Ambiente` e os tipos de acesso são `static class` de constantes (`ConstantesAcessos.cs`), **não enums C#**: o valor gravado no banco precisa ser legível e não virar `int`. Valor fora da lista é rejeitado com `400`.

`ValidadoresAcesso` (novo no projeto — não havia normalizador antes) centraliza CNPJ (14 dígitos, vazio → `null`), IPv4 e faixa de porta. O CNPJ **não é mais gravado na `tbclienteacesso`** (`CAO_Fantasia`/`CAO_Cnpj` foram dropadas — a identidade vem da `tbcliente` por JOIN): a normalização para 14 dígitos vale para a **exibição** e para a comparação no typeahead, com a máscara ficando na apresentação. Unicidade de unidade é **só** por `CLIENTE_ID` (índice único filtrado em `CAO_ClienteId`); IPv4 e faixa de porta continuam valendo para os 5 tipos de acesso.

### Paginação

`GET /api/v1/acessos/clientes` é o **primeiro padrão de paginação do projeto**: envelope `Paginado<T>` (`items`, `totalCount`, `pageNumber`, `pageSize`, `totalPages`, `hasNext`, `hasPrev`) e `PaginadoDefaults.Normalizar` (default 20, máximo 200). Antes, todo endpoint devolvia `List<T>` completo e o frontend filtrava no cliente.

### Armadilha de modelagem — `SenhaCifrada` e `TemSenha` não estão em `AcessoBase`

⚠️ **Não "simplifique" colocando essas propriedades na classe base.** Cada uma das quatro classes com senha (`AcessoRdp`, `AcessoVpn`, `AcessoAnyDesk`, `AcessoBanco`) declara a **própria** `SenhaCifrada` com o `[Column]` da sua tabela, e o **próprio** `TemSenha` (`[NotMapped] => !string.IsNullOrWhiteSpace(SenhaCifrada)`).

Se a base também declarasse, a propriedade da derivada a **sombrearia** e o `TemSenha` herdado leria a propriedade da **base** — que nunca é preenchida — devolvendo sempre `false`, mesmo com a senha gravada. Foi exatamente o bug que motivou a decisão. Há teste de regressão em `backend/Central_BackEnd.Tests/TemSenhaTests.cs`, que também afirma por reflexão que `AcessoBase` e `AcessoActyonWeb` **não** têm `SenhaCifrada`.

⚠️ **Não confunda com `Usuario`.** A base **declara** `Usuario` com `[MaxLength(100)]` mas **sem** `[Column]`, e `AcessoActyonWeb` **não** a redeclara — herda-a, e o EF mapeia por convenção para a coluna `Usuario`, sem prefixo. Essa coluna é **legítima**, está mapeada e foi preservada; a de senha, essa sim, era órfã e foi removida pela migration `20260929013916`. Detalhe em [`04-ESTRUTURA-DADOS.md`](../../arquitetura/banco-dados.md).

