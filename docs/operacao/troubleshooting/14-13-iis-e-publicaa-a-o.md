---
id: sec.14-13-iis-e-publicaa-a-o
titulo: "13. IIS e publicaÃ§Ã£o"
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
tokens_estimados: 3007
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 13. IIS e publicaÃ§Ã£o

### Topologia confirmada

| `-Ambiente` | Camada | Porta | Pasta fÃ­sica no servidor |
|---|---|---:|---|
| `HML` | Frontend | 1010 | `C:\inetpub\wwwroot\Suporte_Front` |
| `HML` | Backend | 1009 | `C:\inetpub\wwwroot\Suporte_Back` |
| `PROD` | Frontend | 1010 | `C:\inetpub\wwwroot\CentralOperacao\FRONT` |
| `PROD` | Backend | 1009 | `C:\inetpub\wwwroot\CentralOperacao\Back` |

O deploy empacota em `deploy/backend` e `deploy/frontend`, mas publica o backend completo e somente `deploy/frontend/browser` no frontend.

### Pre-flight (fase 5b, antes do `app_offline.htm`)

Antes de tocar o IIS, o publish aborta se: houver `app_offline.htm` residual (deploy anterior interrompido); faltar o `appsettings.json` do ambiente no backend remoto; ou faltar `assets\config\runtime-config.json` no frontend remoto (neste caso a mensagem indica a `apiBaseUrl` esperada: `http://<servidor>:1009/api/v1`).

### Comportamentos crÃ­ticos

- `deploy.ps1` usa `npm run build` e `dotnet publish -c Release`, enquanto `validate.ps1` usa `dotnet build` Debug; o package Release Ã© o gate do artefato final.
- O package local usa `robocopy /E`, que copia e atualiza, mas **nÃ£o espelha nem limpa** arquivos antigos em `dist`, `publish_output` ou `deploy`.
- O `appsettings*.json` do servidor Ã© excluÃ­do do mirror (`/XF`) e preservado; se `appsettings.json` faltar no servidor, o pre-flight **aborta** o publish.
- O `runtime-config.json` do servidor Ã© preservado (backup/restauraÃ§Ã£o na fase 8); se faltar, o pre-flight **aborta** â€” no primeiro deploy, crie o arquivo no servidor antes do publish.
- `web.config` **nÃ£o** Ã© excluÃ­do: o backend usa o arquivo gerado pelo `dotnet publish` e o frontend usa o fallback SPA de `frontend/public/web.config`.
- O backend cria `app_offline.htm` antes do mirror. Em falha, o arquivo pode permanecer para impedir inÃ­cio parcial; um rollback feito por cÃ³pia tambÃ©m pode deixÃ¡-lo.
- O frontend precisa do mÃ³dulo IIS URL Rewrite para rotas SPA.
- O `web.config` versionado do backend atualmente habilita Swagger e stdout log; o servidor pode sobrescrever o arquivo no prÃ³ximo publish.
- A publicaÃ§Ã£o monta o compartilhamento administrativo `\\<servidor>\c$` com `New-PSDrive -Credential` (fase 5). Falha nessa fase aborta **antes** de qualquer toque no IIS: o backup (fase 6) e o `app_offline.htm` (fase 7) ficam para depois, entÃ£o o ambiente publicado permanece intacto.

### Sintomas, causas e soluÃ§Ãµes

| Sintoma | Causa provÃ¡vel | SoluÃ§Ã£o |
|---|---|---|
| backend mostra pÃ¡gina `app_offline.htm` | mirror falhou ou publicaÃ§Ã£o ficou parcial | nÃ£o remova o arquivo antes de concluir o mirror; use o procedimento de lock/rollback |
| frontend 404 em rota com ID | URL Rewrite ausente ou `web.config` nÃ£o publicado | restaure o build pelo fluxo de deploy e valide o mÃ³dulo IIS |
| backend 500 sÃ³ no IIS | `web.config`, appsettings preservado, pool, SQL ou ambiente | leia log, recicle e compare a estrutura de configuraÃ§Ã£o sem copiar segredos |
| appsettings sumiu | incidente externo; o script deveria preservÃ¡-lo | interrompa o deploy, restaure config protegida e trate como incidente |
| aviso de `appsettings.json` ausente no pre-flight | configuraÃ§Ã£o do ambiente nÃ£o criada no servidor | crie o `appsettings` do ambiente no servidor antes do publish; o script aborta, nÃ£o continua |
| pre-flight aborta: `runtime-config.json` ausente | primeiro deploy sem o arquivo no servidor | crie `assets\config\runtime-config.json` no servidor com a `apiBaseUrl` do ambiente antes do publish |
| pre-flight aborta: `app_offline.htm` residual | deploy anterior interrompido | resolva manualmente no servidor antes de republicar |
| publish em PROD aborta antes do build | trava: sem `-Backup 1` ou sem digitar `PROD` | publique com `-Backup 1` e confirme digitando `PROD` |
| publish aborta em "Fase 5: Autenticando em ..." com `Erro na autenticacao: "NÃ£o sÃ£o permitidas vÃ¡rias conexÃµes a um servidor ou recurso compartilhado pelo mesmo usuÃ¡rio, usando mais de um nome de usuÃ¡rio"` | sessÃ£o `net use` residual de `\\<servidor>\C$` aberta sob outra credencial: o `New-PSDrive -Credential` do `deploy.ps1` nÃ£o abre uma segunda sessÃ£o para o mesmo servidor | limpe a sessÃ£o e publique de novo: `net use \\<servidor>\C$ /delete /y` (servidor do ambiente: `192.168.2.130` HML, `192.168.2.254` PROD). O aborto Ã© **antes** do IIS â€” o ambiente publicado fica intacto |
| publish aborta em "Fase 5: Autenticando em ..." com `Erro na autenticacao: "Nome de usuÃ¡rio ou senha incorretos"` | credencial do usuÃ¡rio de IIS **rotacionada** no servidor de destino: a senha que o script usava antes nÃ£o autentica mais em PROD (a mesma ainda autentica em HML) | obtenha a senha vigente e passe-a explicitamente em `-SenhaRemota` (o script exige `-SenhaRemota` desde a remoÃ§Ã£o da credencial fixa); nunca reintroduza senha no cÃ³digo ou no repositÃ³rio |
| arquivo obsoleto continua no IIS | package incremental `/E` preservou artefato antigo e o mirror o tratou como parte do pacote | feche processos, limpe somente saÃ­das geradas, refaÃ§a package e valide os artefatos antes do publish |
| `-SenhaRemota` ausente no publish | o script exige credencial explÃ­cita, sem fallback | forneÃ§a `-SenhaRemota` via `Get-Credential`/secret manager e nunca registre o valor |
| rollback deixa backend offline | `app_offline.htm` foi excluÃ­do da cÃ³pia e permanece no destino | conclua e valide o mirror; remova o arquivo apenas depois de confirmar DLL/runtimeconfig e configuraÃ§Ã£o |
| IIS servindo versÃ£o antiga | pool nÃ£o reciclou, URL/endereÃ§o diferente ou cache do navegador | recicle, valide build/hash e faÃ§a recarregamento forÃ§ado sem alterar `BUILD_INFO` |
| 404 de rota API removida | backend publicado antigo ou frontend/cÃ³digo dessincronizado | compare o commit com `BUILD_INFO` e publique o pacote do HEAD esperado |
| stdout logs acumulam | `stdoutLogEnabled=true` no `web.config` atual | capture o necessÃ¡rio, desabilite no cÃ³digo/configuraÃ§Ã£o e remova logs antigos com polÃ­tica aprovada |

### 13.1 Todas as rotas do backend respondem `500` com corpo vazio

**Sintoma.** **Todas** as rotas do backend respondem `500` com **corpo vazio** â€” inclusive rota que **nÃ£o existe** (`/naoexiste` deveria ser `404`). O site inteiro parece quebrado, mas nÃ£o hÃ¡ pista no log.

**Causa.** Schema errado do bloco `<environmentVariables>` no `web.config` do backend. O bloco precisa ser **filho de `<aspNetCore>`** e usar `<environmentVariable name= value=>`:

```xml
<aspNetCore processPath="dotnet" arguments=".\Central_BackEnd.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="WhatsApp__Habilitado" value="true" />
    <environmentVariable name="WhatsApp__BaseUrl" value="http://192.168.2.253:2785/api" />
    <environmentVariable name="WhatsApp__WebhookUrl" value="http://192.168.2.130:1009/api/v1/whatsapp/webhook/openwa" />
    <environmentVariable name="WhatsApp__ApiKey" value="SUA_CHAVE" />
    <environmentVariable name="WhatsApp__WebhookSecret" value="SUA_SEGREDO" />
  </environmentVariables>
</aspNetCore>
```

O erro Ã© escrever o bloco no schema de `applicationHost.config` â€” `<environmentVariables><add key= value=/></environmentVariables>` â€” como **irmÃ£o** de `<aspNetCore>`. O ANCM **nÃ£o sobe o processo**, e o IIS devolve `500` vazio para tudo.

**Por que o diagnÃ³stico engana:**

| Sinal | O que parece | O que Ã© |
|---|---|---|
| Event Log do ANCM (`1012`/`1032`/`1033`) | erro de start do ANCM | evento **normal** (start/recycle/shutdown) |
| `stdout` do app | nenhum log de aplicaÃ§Ã£o | em hosting **in-process** o console nÃ£o Ã© anexado: **a ausÃªncia de log nÃ£o Ã© evidÃªncia** |
| `smoke.ps1` | falha genÃ©rica | detector **confiÃ¡vel**: falha no check **â€œSwagger/APIâ€** |

**Como confirmar (em ordem):**

1. Abra o `web.config` publicado do backend (`C:\inetpub\wwwroot\Suporte_Back\web.config` na HML, `...\CentralOperacao\Back\web.config` na PROD) e confirme que `<environmentVariables>` estÃ¡ **dentro** de `<aspNetCore>` e que cada item usa `name`/`value` (e nÃ£o `key`).
2. Rode `smoke.ps1 -Servidor <servidor-do-ambiente>`: se o frontend passa e o **â€œSwagger/APIâ€** falha, o backend nÃ£o subiu â€” Ã© este o detector do problema.
3. Confirme no `Event Viewer â†’ Applications and Services Logs â†’ Microsoft â†’ IIS â†’ ASP.NET` se hÃ¡ erro do ANCM **diferente** de `1012`/`1032`/`1033`.
4. Corrija o bloco e **recycle**: crie e remova `app_offline.htm` na pasta do backend (o mecanismo usado pelo `deploy.ps1`) ou recicle o application pool.

âš ï¸ **O bloco Ã© regerado a cada publish.** O `deploy.ps1` espelha o backend com `/MIR` excluindo **apenas** `appsettings*.json` e `app_offline.htm`: o `web.config` Ã© sobrescrito e o bloco manual **segue junto**. Com o mÃ³dulo WhatsApp ativo, o bloco precisa ser **reaplicado depois de cada deploy** â€” ver [`deploy.md`](../deploy.md) e [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) Â§7.

### 13.2 Mascote JOTA 3D (`app-jota-3d`): 404.3 em `.glb`, WebGL e fallback 2D

O componente `app-jota-3d` ([`frontend/src/app/shared/components/jota-3d/`](../../../frontend/src/app/shared/components/jota-3d/)) é **presentacional**: recebe o `estado` da tela (`/chat` e widget) e o reflete. O gate `JOTA_3D_HABILITADO` (`jota-3d.types.ts`) está **`false`** — enquanto isso, `three.js` nunca é importado e o comportamento esperado é o **poster 2D** (`assets/images/jota.png`). Gate desligado **não** é defeito de deploy.

| Sintoma | Causa provável | Solução |
|---|---|---|
| `GET /assets/models/jota/jota.glb` (ou `.gltf`/`.bin`/`.ktx2`) responde **404.3** no IIS | extensão sem MIME no `<staticContent>` do `web.config` publicado | o `frontend/public/web.config` versionado já traz os quatro `mimeMap`; confirme o arquivo na raiz do site publicado (`Suporte_Front`/`CentralOperacao\FRONT`) e republice pelo fluxo de deploy — o build copia `public/web.config` para `browser/web.config` |
| mesmo com gate ligado e GLB publicado, o 3D não aparece | WebGL indisponível no cliente, `prefers-reduced-motion: reduce` ou falha ao carregar o GLB | é o fallback projetado: o poster 2D continua visível e o componente emite `(falha)` com motivo `webgl` ou `modelo` — a tela pode registrar o motivo; não é erro do IIS |
| CPU/GPU subindo com a aba oculta ou o componente fora do viewport | loop de render sem pausa | o loader pausa o RAF em `visibilitychange` e por `IntersectionObserver` (threshold 0,05); se voltar a subir, confirme se a versão publicada contém o `jota-3d.loader.ts` atual |
| memória cresce a cada abrir/fechar do widget | dispose incompleto do contexto WebGL | `destruir()` cancela o RAF, desconecta os observers, para as ações do mixer, dá `dispose()` em geometrias/materiais/texturas, chama `renderer.dispose()` e `forceContextLoss()` e remove o canvas; compare com o código publicado |

O GLB (`frontend/src/assets/models/jota/jota.glb`, fase 5) e a ligação do gate (fase 6) são pendências registradas em [`planos/jota-3d/00-PLANO-E-ESFORCO.md`](../../planos/jota-3d/00-PLANO-E-ESFORCO.md) e em [`deploy.md`](../deploy.md) §14. Para diagnosticar/preparar a geração do modelo (estado "modelo inexistente"), use o roteiro [`planos/jota-3d/01-FASE-5-GERACAO-DO-MODELO.md`](../../planos/jota-3d/01-FASE-5-GERACAO-DO-MODELO.md): os 6 recortes de pose de insumo já estão prontos em [`planos/jota-3d/referencias/`](../../planos/jota-3d/referencias/) — falta só gerar/exportar o `.glb` (produção externa). Não há backend nem migration envolvidos na feature.

