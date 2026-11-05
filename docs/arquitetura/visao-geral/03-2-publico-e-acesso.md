---
id: visao-geral.03
titulo: "2. Público e acesso"
tipo: arquitetura
area: produto
sistema: central
tags: [modulos, rotas, ambientes, roadmap]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app/features.routes.ts
  - backend/Central_BackEnd/Program.cs
testes:
  - docs/
tokens_estimados: 483
ficha: docs/modulos/acessos.md
---
> Parte de [01 — Visão geral](./INDEX.md). Ficha: [Acessos Empresas](../../modulos/acessos.md).

## 2. Público e acesso

O público principal é a equipe interna da JCA Soluções que precisa apoiar atendimento, consultar conhecimento, acompanhar implantação e usar ferramentas operacionais.

- `/login` é a única entrada de frontend sem `authGuard`.
- O shell principal exige usuário autenticado.
- `/admin` exige `authGuard` e `adminGuard`; o guard aceita somente o perfil `Administrador`.
- As leituras dos cadastros de tipos e colunas são autenticadas; suas escritas exigem role `Administrador`, e o dashboard administrativo também exige essa role.
- Acesso a dados sensíveis de empresas exige validação de senha, e a prova de senha é um ticket de acesso de 2 minutos — reutilizável para visualizar na mesma empresa, de uso único em cada escrita ou revelação —, com registro de auditoria inclusive em cada revelação de credencial.
- O frontend não é a fronteira final de segurança: controllers e serviços do backend também validam token, role e estado do usuário.

O tipo `PerfilUsuario` do frontend declara `Usuario`, `Editor`, `Administrador`, `Suporte`, `Financeiro` e `Produtor`. O mapper do backend (`AuthService.MapearPerfil`) resolve `SE_ADMIN = 1` → `Administrador` e, por `PERFIL_ID`, `A` → `Administrador`, `S` → `Suporte`, `F` → `Financeiro`, `P` → `Produtor`, com `Usuario` como fallback. **O rótulo é apenas apresentação:** a permissão por função vem do cargo `GESTOR_MASTER` e de `tbgestor` em `FuncaoAuthorizationService`, e o `Usuario` do frontend carrega `perfilId`, `funcaoId`, `funcaoDescricao`, `funcaoClassificacao`, `ehAdmin`, `cargoCodigo` e `cargoId` para reproduzir a regra na UI.

