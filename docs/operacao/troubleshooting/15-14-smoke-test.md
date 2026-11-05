---
id: sec.15-14-smoke-test
titulo: "14. Smoke test"
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
tokens_estimados: 737
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 14. Smoke test

O [`scripts/smoke.ps1`](../../../scripts/smoke.ps1) testa quatro checks (todos obrigatórios para saída 0):

- `http://<servidor>:1010` (frontend)
- `http://<servidor>:1009/swagger/v1/swagger.json` (com `openapi` e `paths` não vazio)
- `http://<servidor>:1010/assets/config/runtime-config.json` (com `apiBaseUrl` igual à do ambiente, ex. `http://<servidor>:1009/api/v1`)
- `POST http://<servidor>:1009/api/v1/auth/login` com usuário fictício, exigindo **HTTP 401** (500 ali é schema/código quebrado)

ParÃ¢metros padrÃ£o: 5 tentativas, 3 segundos entre tentativas e timeout de 10 segundos por requisiÃ§Ã£o. Uma resposta entre 200 e 399 conta como sucesso nos dois primeiros checks; o terceiro exige igualdade exata da `apiBaseUrl` e o quarto exige exatamente `401`.

> **Resolvido (commit `efedfd5`):** o check do Swagger usava `$json.paths.Count`, que em PowerShell 5.1 Ã© sempre vazio porque `paths` Ã© objeto (dicionÃ¡rio), nÃ£o array â€” todo swagger vÃ¡lido falhava. Agora conta `@($json.paths.PSObject.Properties).Count`.

Limites reais:

- nÃ£o faz login;
- nÃ£o envia Bearer;
- nÃ£o acessa `cc_refresh`;
- nÃ£o chama `/api/v1/acessos/*` nem qualquer API de negÃ³cio;
- nÃ£o valida migrations, SQL, Acessos Empresas ou Database Explorer;
- nÃ£o confere o hash do `BUILD_INFO` no servidor;
- nÃ£o garante que a pÃ¡gina carregou o bundle esperado;
- com `-Local`, testa ainda as portas IIS 1010/1009, nÃ£o `ng serve` na 4200.

| Resultado | DiagnÃ³stico |
|---|---|
| frontend, Swagger e runtime-config OK | a disponibilidade bÃ¡sica passou; ainda faÃ§a QA funcional |
| frontend passa, Swagger falha | backend, binding 1009 ou configuraÃ§Ã£o efetiva do Swagger |
| runtime-config WARN (aponta para outro ambiente) | front publicado com a `apiBaseUrl` errada â€” geralmente o `localhost` do pacote no primeiro deploy; corrija o arquivo no servidor e repita o publish |
| runtime-config WARN (nÃ£o foi possÃ­vel ler) | arquivo ausente no servidor ou frontend fora do ar; o pre-flight tambÃ©m aborta o publish sem ele |
| ambos falham | conectividade, bindings, pools, web.config ou IIS |
| smoke passa, tela autenticada falha | CORS, cookie, JWT ou API de negÃ³cio nÃ£o cobertos pelo script |
| smoke falha apÃ³s a publicaÃ§Ã£o | aguarde 30â€“60 s, repita e entÃ£o diagnostique IIS/logs |

Como o script depende de `/swagger`, uma polÃ­tica que desabilite Swagger torna o smoke incompatÃ­vel. Nesse caso, crie um endpoint de verificaÃ§Ã£o de saÃºde autorizado e adapte o script; nÃ£o declare o pipeline verde apenas ignorando a falha.

