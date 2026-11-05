---
id: sec.09-8-cors
titulo: "8. CORS"
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
tokens_estimados: 399
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 8. CORS

A polÃ­tica `Angular` permite exatamente (fontes: `Cors:AllowedOrigins` em [`appsettings.json`](../../../backend/Central_BackEnd/appsettings.json), lida pelo `Program.cs`):

- `http://192.168.2.130:1010` (HML)
- `http://192.168.2.254:1010` (PROD â€” frontend no IIS PROD)
- `http://localhost:4200`
- `http://localhost:1010`

Ela permite headers e mÃ©todos e habilita credenciais. NÃ£o hÃ¡ wildcard.

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| bloqueio CORS no console | origem nÃ£o estÃ¡ na lista | use uma origem exata ou altere `Program.cs`, valide e publique o backend |
| login funciona, refresh falha | polÃ­tica/cookie nÃ£o atravessa o navegador | confira `Access-Control-Allow-Credentials`, preflight e `withCredentials` |
| funciona em `localhost`, falha no IP | mistura de protocolo, host ou porta | compare a origem completa; portas diferentes geram outra origem |
| funciona com 4200, falha com 4201 | apenas 4200 estÃ¡ autorizada | nÃ£o troque a porta informalmente; adicione a origem de forma revisada |
| cookies nÃ£o funcionam com hostname novo | `SameSite=Strict` e polÃ­tica atual | teste a topologia planejada e ajuste cÃ³digo/configuraÃ§Ã£o antes do deploy |

