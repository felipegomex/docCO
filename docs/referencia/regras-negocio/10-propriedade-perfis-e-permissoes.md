---
id: regras-negocio.10
titulo: "Propriedade, perfis e permissões"
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
tokens_estimados: 1233
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

## Propriedade, perfis e permissões

**Situação: parcial; há autenticação, mas não há propriedade uniforme.**

- Os controllers de Agenda, Acessos, Chat (`/api/chat`), Conhecimento (`/api/conhecimento`), Banco de Dados, Projetos, Tarefas e Dashboard exigem Bearer, exceto o callback OAuth do Gitea, explicitamente anônimo. Chat e Conhecimento são exceções de versionamento: não usam `/api/v1`. Tipos de projeto e colunas exigem Bearer; as escritas do **catálogo** de colunas exigem o papel `Administrador`, e a **visão do setor** (`colunas-kanban/visao/{funcaoId}`) exige o gestor da função ou o administrador. O dashboard administrativo exige `Administrador` no backend. No Dashboard Integrado, comercial (`/api/v1/dashboard/comercial`) e administrativo (`/api/v1/dashboard/administrativo`) exigem apenas Bearer, enquanto o financeiro (`/api/v1/dashboard/financeiro`) exige `Administrador` — no frontend, a rota `/dashboard/financeiro` também aplica `adminGuard`.
- `AuthService.MapearPerfil` resolve `SeAdmin = true` para `Administrador` e, por `PERFIL_ID`, `A` → `Administrador`, `S` → `Suporte`, `F` → `Financeiro` e `P` → `Produtor`, com `Usuario` como fallback. **O rótulo é apresentação.** O admin legado é decidido por `SE_ADMIN`/`PERFIL_ID = 'A'`; a permissão por função do Kanban vem do cargo `GESTOR_MASTER` + `tbgestor` (ver `FuncaoAuthorizationService`).
- **`PERFIL_ID` é lido em um lugar só:** [`PerfilOperador`](../../../backend/Central_BackEnd/Services/Shared/PerfilOperador.cs) normaliza (trim + maiúsculas), decide admin (`SE_ADMIN` ou letra `A`) e produz o rótulo. A coluna é `varchar(1)` com **cadastro manual** — nada impede `"a"` minúsculo ou `" A "` com espaço — e cada um normalizava do seu jeito: o Kanban (`FuncaoAuthorizationService`) normalizava e os apontamentos (`TarefaService`) comparavam cru, então o mesmo operador era **admin no Kanban** e **não-admin nos apontamentos**. Agora o token carrega a letra **normalizada** (`perfil_id`) e todos os consumidores leem a mesma regra.
- `CriarAsync` e `AtualizarAsync` verificam o responsável principal (`ResponsavelId` ou o primeiro item de `ResponsavelIds` quando o principal é omitido): se for diferente do operador autenticado, chamam `PodeAtribuirTerceiroAsync`.
- A lista `ResponsavelIds` não tem uma regra uniforme: na criação, cada item é apenas consultado quanto à existência e somente os existentes são adicionados, sem a autorização de atribuição usada no responsável principal; na atualização, quando a lista é enviada, cada item diferente do operador passa pela mesma autorização de atribuição. Se a lista é omitida, a autorização considera apenas o caminho do responsável principal. Portanto, não se deve afirmar que todos os responsáveis adicionais são igualmente verificados em ambos os fluxos.
- A autorização de atribuição permite o próprio operador, o `GESTOR_MASTER` e o gestor da função alvo (`tbgestor`, sem atribuir a outro gestor); não é uma autorização de propriedade do recurso.
- Em Agenda, o responsável ou administrador pode alterar, mover ou excluir o evento. A transferência de responsabilidade para outro operador é negada para não administrador, salvo se o destino for o próprio usuário.
- Não foi localizada regra de propriedade para projeto, tarefa, apontamento, comentário, vínculo de chamado ou exclusão de tarefa. Em tarefas, a checagem de responsável é a autorização de atribuição descrita acima, não uma autorização de acesso ou propriedade; um usuário autenticado pode alcançar as demais operações se passar pelas validações do controller.
- `ProjetoService` não valida o `ResponsavelId` enviado no cadastro/atualização; a referência é persistida como identificador lógico.
- Vários campos de identidade vêm do payload quando não estão vazios: `CriadorId`, `UsuarioAlteracao` e `AutorId` podem ser enviados pelo cliente. O servidor não os força de forma universal ao claim atual.
- O perfil `F` **não** é mais superusuário do Kanban: `EhPerfilF`/`EhSuporte` são `false` e a autoridade vem do cargo `GESTOR_MASTER` ou de `tbgestor`. A atribuição de terceiros segue a mesma fonte (Master ou gestor da função alvo).
- No frontend, `AuthService.hasRole` compara **o rótulo e a letra** de `PERFIL_ID` (`user.perfilId`, claim `perfil_id` — o backend já a emite normalizada), e é por isso que o menu pode ser escrito como `roles: ['F']` mesmo depois do rename do rótulo para `Financeiro`. `header-nav.config.ts` declara `NavRole = PerfilUsuario | 'A' | 'S' | 'F' | 'P'` para esse fim.
- `adminGuard` é proteção de navegação no frontend; a proteção efetiva continua sendo os `[Authorize]` do backend.

