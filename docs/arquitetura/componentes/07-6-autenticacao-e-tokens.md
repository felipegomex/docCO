---
id: componentes.07
titulo: "6. Autenticação e tokens"
tipo: arquitetura
area: arquitetura
sistema: central
tags: [arquitetura, componentes, jwt, cors]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 594
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 6. Autenticação e tokens

### Access token

- É um JWT HMAC SHA-256, validado por issuer, audience, assinatura, algoritmo, lifetime e expiração.
- Os claims carregam subject, role, nome, email, perfil, `jti` e dados da função quando disponíveis.
- O tempo configurado para o access token é `Jwt:AccessTokenMinutes=240`, isto é, quatro horas.
- O frontend o mantém em uma propriedade privada de `TokenStorageService`, portanto não o grava em `localStorage` nem em `sessionStorage`.

### Refresh token rotativo

- O valor é aleatório, tem 64 bytes de entropia e é armazenado no backend apenas como hash SHA-256.
- Cada token recebe `Jwt:RefreshTokenHours=4`, ou seja, quatro horas desde sua emissão.
- O refresh marca o token anterior como revogado e grava `SubstituidoPor` com o hash do novo token.
- A rotação atual não é atômica: não há transação nem atualização condicional explícita que impeça duas requisições concorrentes de usarem o mesmo token antes da revogação.
- O controller grava o novo valor em `cc_refresh`; o JSON retornado ao frontend não contém o segredo.
- `LembrarAcesso=true` define `MaxAge` de quatro horas para `cc_refresh`; `cc_lembrar` é acrescentado sem `MaxAge` e permanece um cookie de sessão. Sem essa opção, o cookie de refresh também é de sessão do navegador; a validade do token no servidor continua sendo de quatro horas.
- O frontend restaura a sessão por refresh no startup, após `401` e na verificação periódica a cada cinco minutos. Como cada refresh emite outro token de quatro horas, a sessão ativa tem renovação deslizante.

### Cookie e transporte

`cc_refresh` é criado com `HttpOnly=true`, `SameSite=Strict`, `Path=/` e `Secure=false`. `cc_lembrar`, quando presente, usa `HttpOnly=false` e armazena somente a preferência `1`.

O interceptor envia `withCredentials=true` para permitir o cookie cross-origin entre os sites configurados. Como `Secure=false` e as URLs de ambiente são HTTP, o cookie não depende de HTTPS para ser aceito; essa é uma limitação de transporte, não uma recomendação de configuração.

