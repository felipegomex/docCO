---
id: sec.04-3-angular-serve-e-build
titulo: "3. Angular: serve e build"
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
tokens_estimados: 641
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 3. Angular: serve e build

### Desenvolvimento

```powershell
Set-Location .\frontend
npm ci
npm start
```

A URL esperada Ã© `http://localhost:4200`. O frontend chama a API diretamente; nÃ£o existe `proxy.conf.json` no projeto.

### Build de produÃ§Ã£o

```powershell
Set-Location .\frontend
npm run build
```

Artefatos exigidos pelo deploy:

- `frontend/dist/central-conhecimento-actyon/browser/index.html`

O script publica no IIS somente a pasta `browser`. Uma pasta `server/` no pacote gera aviso e nÃ£o Ã© publicada.

### DiagnÃ³stico

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| `Port 4200 is already in use` | outro `ng serve` ou aplicaÃ§Ã£o ocupa a porta | Identifique o processo pelo `CommandLine` e encerre apenas o processo do projeto. Se usar outra porta, inclua a nova origem no CORS do backend |
| tela carrega, mas toda API falha em CORS | origem nÃ£o estÃ¡ na lista do backend | Use exatamente uma origem autorizada e confira protocolo, host e porta |
| tela branca e console mostra exceÃ§Ã£o | erro JavaScript/runtime | Abra DevTools, leia o primeiro erro, confirme `apiBaseUrl` e valide novamente |
| build falha por TypeScript | API do Angular/TypeScript alterada ou dependÃªncia incompatÃ­vel | Rode `npm ci`, use a versÃ£o de Node do projeto e corrija o primeiro erro do compilador |
| build falha por budget | bundle inicial acima de 2 MB ou estilo de componente acima de 25 kB | NÃ£o ignore o budget; remova peso/duplicaÃ§Ã£o ou ajuste o limite somente com revisÃ£o tÃ©cnica |
| `browser/index.html` ausente | build incompleto/interrompido ou foi usado apenas outro comando | Feche builds concorrentes e rode `npm run build` por completo |
| rota com parÃ¢metro retorna 404 apÃ³s Ctrl+F5 no IIS | fallback SPA ausente | Confirme `frontend/public/web.config` no build e o mÃ³dulo URL Rewrite no IIS |
| CSS parece reduzido em quase todas as telas | escala global intencional de 80% | Ã‰ esperado `html.scaled`; `/login` permanece em 100%. NÃ£o confunda com zoom do navegador |

Avisos de budget nÃ£o sÃ£o iguais a erro: o pipeline prossegue apenas quando o processo termina com cÃ³digo 0 e o `validate.ps1` retorna `VALIDATE: OK`.

