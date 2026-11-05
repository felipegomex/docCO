---
id: regras-negocio.03
titulo: "Autenticação e refresh"
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
tokens_estimados: 603
ficha: docs/modulos/auth-rbac-sistema.md
---
> Parte de [Regras de negócio](./INDEX.md). Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

## Autenticação e refresh

**Situação: implementado; proteção de transporte e identidade parcialmente implementada.**

- `POST /api/v1/auth/login` recebe `Usuario`, `Senha` e `LembrarAcesso`. O serviço aceita identificador de operador ou e-mail; rejeita operador inexistente, inativo, senha inválida ou chave bloqueada pelo controle de tentativas.
- O token de acesso JWT é emitido com expiração lida de `Jwt:AccessTokenMinutes`. O refresh token é aleatório, tem expiração lida de `Jwt:RefreshTokenHours` e é armazenado no banco somente como hash SHA-256.
- O refresh gira o token: o anterior é revogado, recebe `SubstituidoPor` com o hash do novo e o novo é emitido. Token revogado, expirado ou associado a operador inativo não renova.
- O controller envia o refresh token no cookie `cc_refresh` com `HttpOnly`, `SameSite=Strict` e `Path=/`; o token não é removido do JSON: os controllers devolvem `refreshToken: ""` no login e no refresh. `LembrarAcesso=true` acrescenta `MaxAge` de quatro horas; sem essa opção, o cookie fica sem `MaxAge`.
- No frontend, o token de acesso e o usuário ficam somente em memória; o refresh token não é gravado em `localStorage`/`sessionStorage`. As requisições usam `withCredentials` para enviar o cookie.
- `POST /api/v1/auth/refresh` não exige token de acesso no controller. O frontend envia um objeto vazio, mas o backend não lê o corpo. `POST /api/v1/auth/logout` exige Bearer, revoga o cookie quando presente e devolve `204`.
- A comparação de senha do login e da revalidação de Acessos aplica SHA-256 ao valor informado e ao valor lido do operador; não há, no código consultado, migração para um algoritmo de hash adaptativo com sal.
- O `BruteForceGuard` é singleton em memória: cinco falhas na janela de cinco minutos bloqueiam a chave por quinze minutos. O estado não é distribuído entre processos.
- O cookie de refresh está com `Secure=false` no código atual. HSTS só é habilitado fora de Development e não há, neste arquivo, uma regra explícita de redirecionamento para HTTPS. Isso deve ser corrigido antes de exposição por rede não confiável.

