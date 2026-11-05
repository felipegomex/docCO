---
id: 06-5-modelos-dbcontext-e-relacionamentos.03
titulo: "5.2.1 Módulo de Ideias — estado em 2026-10-12"
tipo: referencia
area: backend
sistema: central
tags: [backend, modelos, dbcontext, efcore]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 990
ficha: docs/modulos/ideias.md
---
> Parte de [5. Modelos, DbContext e relacionamentos](./INDEX.md). Ficha: [Ideias, Melhorias e Erros](../../../modulos/ideias.md).

### 5.2.1 Módulo de Ideias — estado em 2026-10-12

**Implementado de ponta a ponta.** Persistência, autorização, serviço de negócio, controllers e tela existem.

| Camada | Arquivo | Situação |
|---|---|---|
| Entidades | `Models/Ideias/*.cs`, `Models/Gestores/Gestor.cs` | ✅ F1 |
| Autorização | `Services/Ideias/IdeiasPermissaoService.cs`, `Services/Gestores/GestorService.cs` | ✅ F2 |
| Serviço de negócio | `Services/Ideias/IdeiaService.cs` | ✅ F5 — inclui `ExcluirAsync` (transacional: apaga satélites e a solicitação; tarefa do Kanban sobrevive) e `PodeExcluirAsync` (admin, validado no controller, ou gestor geral/da função do criador; autor sem gestão não exclui); auditoria grava "Solicitacao excluida" |
| Controller | `Controllers/IdeiasController.cs`, `Controllers/GestoresController.cs` | ✅ F5/F11 — 16 operações em Ideias (incluindo `DELETE /{id}` admin ou gestor da ideia) e 6 em Gestores |

**A autorização não reutiliza `IFuncaoAuthorizationService`.** É a decisão que mais importa neste módulo: aquele serviço decide **permissão por função** (Master + `tbgestor`), enquanto Ideias precisa do **papel sobre a solicitação** (criador, gestor da função do criador, administrador da ideia). Delegar a ele misturaria os dois eixos e ampliaria a leitura para o setor inteiro. Em Ideias, a fonte única é **`tbgestor`**, e `IdeiasPermissaoService` não consulta perfil.

O papel sai da função do **criador** (`tboperador.FUNCAO_ID`): o gestor daquela função responde pelo pedido; o gestor geral (`GES_FuncaoId` NULL) responde por todas.

A permissão por operação e fase é uma tabela só — `IdeiasPermissaoService.Matriz`, 13 operações × 7 status — consultada por `PermissaoIdeia.PodeOperar`. Um teste garante que toda operação tem linha para todo status, para que uma operação nova não negue acesso a todo mundo em silêncio.

**Leitor do setor CIAA (2026-10-27).** `PapelIdeia.Leitor = 3` é atribuído quando o operador e o criador pertencem a `FUNCAO_ID = 5` (`IdeiasPermissaoService.FuncaoCiaa`). O leitor entra somente no conjunto de `Ler`, em todas as fases: a matriz não concede `Comentar`, `EditarCampos`, `ResponderEsclarecimento`, `Transicionar` nem `Aprovar`. `EhLeitorCiaaAsync` consulta a função do operador; `IdeiaService.ListarAsync` inclui as solicitações cujo criador é do CIAA quando o operador também é do CIAA (além das próprias e das funções geridas), e `MontarResumo` trata `Gestor` **e** `Leitor` como aptos a ver o nome do criador. O controller devolve `ehLeitor` e `podeVerCriador = ehGestor || ehLeitor` em `GET /ideias/minhas-permissoes`, e o aviso de "não administra" não aparece para o leitor.

Cobertura: suíte com filtro Ideias em **181 verdes** — inclui os 5 testes da exclusão em `IdeiasCicloDeVidaTests.cs` (apaga satélites; inexistente devolve false; service não consulta a matriz, a barreira é o controller; `PodeExcluir`: gestor pode, autor e terceiro não; tarefa vinculada sobrevive). Em 2026-10-27 a suíte ganhou `IdeiasCiaaLeitorTests.cs` (**10** casos), `PermissoesCargoHierarquiaTests.cs` (**8**) e, na Fase 6 do plano do WhatsApp, **12 casos** nos quatro arquivos de protocolo/finalização/foto/métricas; `dotnet test` fechou em **921 aprovados**.

Schema, colunas e índices em [`04-ESTRUTURA-DADOS.md`](../../../arquitetura/banco-dados.md); cenários em [`referencia/qa-historias.md`](../../qa-historias.md) §10.

Configuração já reservada em `appsettings.json` (sem segredo, tudo desligado por padrão):

```json
"Ideias": {
  "Seed": { "Habilitado": false, "ConfirmacaoBanco": "" },
  "Ia":    { "Habilitada": false, "ChaveApi": "", "Modelo": "gemini-2.0-flash", "TimeoutSeconds": 60 }
}
```

