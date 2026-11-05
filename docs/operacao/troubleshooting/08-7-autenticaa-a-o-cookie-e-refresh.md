---
id: sec.08-7-autenticaa-a-o-cookie-e-refresh
titulo: "7. AutenticaÃ§Ã£o, cookie e refresh"
tipo: operacao
area: diagnostico
sistema: central
tags: [erro, diagnostico]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts
  - backend/Central_BackEnd
testes:
  - docs/
tokens_estimados: 1315
ficha: docs/modulos/auth-rbac-sistema.md
---
> Ficha: [Auth, RBAC e Sistema](../../modulos/auth-rbac-sistema.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 7. AutenticaÃ§Ã£o, cookie e refresh

Fluxo confirmado no frontend e backend:

1. O access token e o usuÃ¡rio ficam **somente em memÃ³ria**; recarregar a pÃ¡gina perde ambos.
2. O refresh token fica no cookie HttpOnly `cc_refresh`; nÃ£o deve aparecer em `localStorage` nem no corpo da resposta.
3. O interceptor aplica `withCredentials: true` a todas as requisiÃ§Ãµes.
4. O interceptor exclui URLs que contenham `/auth/` do acrÃ©scimo do header Bearer.
5. Para uma API nÃ£o-auth, um 401 dispara refresh single-flight e repete a requisiÃ§Ã£o original uma vez.
6. O guard tenta refresh silencioso no carregamento da rota.
7. `POST /api/v1/auth/logout` exige Bearer, mas Ã© excluÃ­do do header pelo interceptor. Com o comportamento atual, o logout pode receber 401; `AuthService.logout` engole o erro, limpa apenas o estado local e pode manter o cookie/refresh token ativo no servidor.

### Endpoints de autenticaÃ§Ã£o

| MÃ©todo | Rota | AutorizaÃ§Ã£o | Corpo/uso |
|---|---|---|---|
| POST | `/api/v1/auth/login` | NÃ£o | `{ "usuario": "...", "senha": "...", "lembrarAcesso": true }` |
| POST | `/api/v1/auth/refresh` | NÃ£o; exige cookie `cc_refresh` | Front envia `{}`; resposta contÃ©m novo access token e usuÃ¡rio |
| POST | `/api/v1/auth/logout` | Bearer | corpo `{}`; somente uma chamada autenticada revoga e limpa cookies; o interceptor atual pode enviar sem Bearer |
| GET | `/api/v1/auth/me` | Bearer | sem corpo |

Propriedades atuais no cÃ³digo: `HttpOnly=true`, `SameSite=Strict`, `Secure=false`, `Path=/`; com â€œlembrar acessoâ€, hÃ¡ `MaxAge` de 4 horas. A configuraÃ§Ã£o preservada no servidor pode alterar a duraÃ§Ã£o efetiva.

### Sintomas, causas e soluÃ§Ãµes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| login 400 | `usuario` ou `senha` vazio | revise o payload da tela; nÃ£o registre o valor |
| login 401 | credencial invÃ¡lida, usuÃ¡rio inativo ou bloqueio temporÃ¡rio | confirme provider/banco corretos; aguarde o lockout quando aplicÃ¡vel |
| login 429 | rate limit | aguarde a janela; nÃ£o automatize tentativas repetidas |
| login funciona usando o valor que estava gravado na coluna `SENHA` do operador | `SegurancaHelper.SenhasIguais` hasheia os dois lados, entÃ£o o valor gravado em `tboperador.SENHA` Ã© a prÃ³pria senha | rotacione a senha do operador e trate a leitura da tabela como comprometimento; nÃ£o reproduza o valor em log, ticket ou documento |
| login 200, mas nenhum `Set-Cookie: cc_refresh` | resposta nÃ£o chegou ao navegador, CORS bloqueou ou cookie foi descartado | inspecione Network, status CORS e atributos do cookie |
| logout redireciona, mas `/auth/logout` retorna 401 e o cookie permanece | o interceptor exclui `/auth/` do Bearer, embora o controller exija `[Authorize]` | registre o status sem copiar token; limpe os dados/cookies do site como mitigaÃ§Ã£o local, trate o refresh token como potencialmente ativo e solicite correÃ§Ã£o do interceptor para revogaÃ§Ã£o server-side |
| refresh sempre 401 | cookie ausente, expirado, revogado ou de origem incompatÃ­vel | verifique `cc_refresh` em Storage/Network e teste `POST /api/v1/auth/refresh` no mesmo contexto |
| F5 volta ao login | access token foi perdido e o cookie nÃ£o foi enviado | corrija CORS/origem/cookie antes de investigar o guard |
| cookie nÃ£o aparece em `document.cookie` | ele Ã© HttpOnly | isso Ã© esperado; inspecione DevTools |
| muitas renovaÃ§Ãµes simultÃ¢neas | vÃ¡rias requisiÃ§Ãµes chegam com 401 juntas | o frontend usa single-flight; confirme que sÃ³ a versÃ£o publicada contÃ©m o interceptor atual |
| access token invÃ¡lido apÃ³s a reciclagem do IIS | a chave JWT do processo mudou | mantenha a mesma chave protegida entre reciclagem/deploy ou force nova autenticaÃ§Ã£o de forma controlada |
| `Set-Cookie` aparece sem o atributo `Secure` | o cÃ³digo usa `Secure=false` | trate como risco em transporte HTTP; migre para HTTPS somente apÃ³s alinhar CORS, bindings e `Secure` |
| chave JWT funciona no startup, mas login falha | a chave efetiva do processo nÃ£o Ã© a esperada (divergente entre instÃ¢ncias ou placeholder): a emissÃ£o e a validaÃ§Ã£o jÃ¡ usam a mesma fonte desde `57a7e4f` | confira a linha `JWT: chave resolvida de {Origem}` do startup; se apontar placeholder, defina `JWT_KEY` no ambiente da **mÃ¡quina** e recicle o pool |
| chave JWT diverge entre instÃ¢ncias (login funciona em uma, falha em outra) | `JWT_KEY` e `Jwt:Key` apontam para valores diferentes; `JWT_KEY` vence | alinhe os dois valores no ambiente da mÃ¡quina; o startup avisa a divergÃªncia |
| cookie deixa de funcionar apÃ³s HTTPS | `Secure=false` e/origem CORS atual nÃ£o contemplate HTTPS | alinhe CORS, `Secure`, URL do frontend e binding IIS antes de migrar para HTTPS |

Ao investigar, nunca copie o conteÃºdo de `cc_refresh`, access token ou senha para o console. Registre apenas presenÃ§a, atributos, origem e status HTTP.

