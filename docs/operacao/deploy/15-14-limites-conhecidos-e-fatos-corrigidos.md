---
id: sec.15-14-limites-conhecidos-e-fatos-corrigidos
titulo: "14. Limites conhecidos e fatos corrigidos"
tipo: operacao
area: deploy
sistema: central
tags: [deploy, iis, migration]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - scripts/deploy
testes:
  - docs/
tokens_estimados: 2524
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/deploy.md](../deploy.md).

# 14. Limites conhecidos e fatos corrigidos

1. `smoke.ps1` testa quatro checks: frontend 1010, `/swagger/v1/swagger.json` (com `openapi`/`paths`), `runtime-config.json` com a `apiBaseUrl` do ambiente e **login fictício exigindo 401** (500 ali indica schema/código quebrado); não faz login válido nem API de negócio. (O falso-positivo antigo do swagger — `$json.paths.Count` sempre vazio em PS 5.1 porque `paths` é objeto, não array — foi corrigido contando `PSObject.Properties`.)
2. O smoke depende de `/swagger/v1/swagger.json` (HTTP 200 + JSON com `openapi` e `paths`); desabilitar Swagger ou remover a validação do JSON torna o gate vermelho.
3. A baseline versionada habilita Swagger no `web.config`; não assuma o padrão antigo “produção false” sem verificar a configuração efetiva do servidor.
4. `validate.ps1` compila o backend em Debug; `deploy.ps1` publica o backend em Release.
5. O package usa `/E` e não limpa artefatos antigos; o mirror `/MIR` ocorre somente no IIS.
6. A ausência de `appsettings.json` no servidor **aborta** o publish no pre-flight (fase 5b), antes do `app_offline.htm`.
7. Um rollback pode deixar `app_offline.htm`; remova-o somente após mirror e artefatos íntegros.
8. O interceptor exclui `/auth/` do Bearer; logout pode receber 401 e não revogar o refresh token.
9. O cookie `cc_refresh` está com `Secure=false`; a chave JWT tem fonte única desde `57a7e4f`, mas `JWT_KEY` no ambiente do *application pool* é apagada a cada recycle e derruba as sessões — use o ambiente da máquina.
10. Chat e Conhecimento usam URLs relativas sem versão e dependem do encaminhamento IIS/reverse proxy não versionado; o Chat envia dados a AnythingLLM/Claude e não tem fallback mock.
11. `BUILD_INFO` compara apenas o hash registrado com o HEAD; não garante árvore limpa nem integridade binária.
12. O package correto é `deploy/frontend/browser`, não `dist/frontend/browser`.
13. O deploy IIS do frontend publica somente `browser`; a aplicação é somente CSR e não há processo Node no site.
14. `appsettings*.json` e `runtime-config.json` do servidor são preservados; `web.config` não.
15. O script **exige** `-SenhaRemota` explícito e aborta sem ele; não há credencial fixa nem fallback no código.
16. O agregado `AllMigrations.sql` foi **apagado** (era obsoleto); gere e revise SQL a partir do commit atual.
17. O script de migrations não existe no deploy; migration é etapa manual e auditável. Schema e binário podem divergir em silêncio: o `smoke.ps1` não cobre API de negócio.
18. Em Development, a flag atual seleciona SQL Homolog; InMemory só ocorre quando alterada para `false`.
19. O resumo do script pode exibir o caminho do backend com prefixo duplicado; valide sempre as pastas reais do IIS.
20. O banco legado do Dashboard Integrado está em `compatibility_level 100` (SQL Server 2008): o smoke **não** cobre nenhuma API de negócio, então uma query com `OPENJSON` pode passar pelo pipeline e só falhar em produção. A mesma restrição vale para o módulo Acessos Empresas.
21. O `validate.ps1` executa `dotnet test` no passo **5/6** e `ng test --watch=false` no passo **6/6** (suíte de `backend/Central_BackEnd.Tests` e specs do frontend via Karma/ChromeHeadless). A medição local de 2026-10-28 fechou em **944 casos de backend, 0 falhas** e **186 testes de frontend, 0 falhas**. Se qualquer um falhar, o gate fica vermelho e o commit/deploy não deve seguir.
22. O módulo Acessos Empresas grava credenciais novas com `CifradorAesGcm` (AES-256-GCM). Linhas antigas sem o prefixo `v1:` continuam legíveis por compatibilidade e podem permanecer em texto puro até o backfill; `GET /acessos/estado-cifra` mostra a implementação, não a cobertura. `tboperador.SENHA` (`varchar(10)`, texto claro) está fora do alcance.
23. Não existe staging: `dbBUSINESS_HML` é o banco de produção. Toda migration é mudança de schema em produção de primeira vez.
24. A conversão Setor → Função (`20260929221556_SetorParaFuncao`) tem **duas fases** com janelas distintas: a Fase 0 roda **antes** do deploy e a Fase 3 **depois**. Inverter a ordem derruba o Kanban Geral ou deixa o schema inconsistente. O selo em `__EFMigrationsHistory` é gravado pela Fase 3, não pela Fase 0 — ausência dele com o código novo no ar significa que a consolidação ainda não rodou.
25. A migração de grupos do Acessos (`20260930132443_AcessosGrupoClienteErp`) tem **janela de incompatibilidade ao contrário da Setor → Função**: o script roda **antes** do publish e não pode esperar. Depois do repoint da FK, o código antigo grava ids de `tbgrupoacesso` em `CAO_GrupoId` apontando para `tbcliente_grupo` (a FK é `NO CHECK`, então o erro não aparece na gravação — aparece como grupo fantasma na tela). Separe script e publish e você terá que auditar dados à mão. **Em 2026-09-30 a janela foi cumprida na ordem correta** no deploy do commit `d646b19` (script → publish → smoke); o registro vale para outros ambientes.
26. Deploy multi-ambiente (commit `efedfd5`): `-Ambiente HML|PROD` resolve servidor + pastas pelo mapa (seção 4); sem ele, o script publica nos defaults de HML com aviso. `PROD` trava sem `-Backup 1` e sem digitar `PROD`, antes de qualquer build. O pre-flight (fase 5b) aborta antes do `app_offline.htm` se houver offline residual ou se faltar `appsettings.json`/`runtime-config.json` no servidor. `BUILD_INFO` grava `Ambiente`/`Servidor`/destinos e cada package/publish deixa recibo em `deploy/history/`.
27. A publicação monta o compartilhamento administrativo `\\<servidor>\c$` com `New-PSDrive -Credential` na **fase 5** e aborta **antes** de tocar no IIS (o backup é fase 6 e o `app_offline.htm` é fase 7): o ambiente publicado fica intacto. Duas falhas distintas, com mensagens distintas (verificadas em 2026-10-20 no publish em PROD): (a) *"Não são permitidas várias conexões a um servidor ou recurso compartilhado pelo mesmo usuário, usando mais de um nome de usuário"* — sessão `net use` residual de `\\<servidor>\C$` sob outra credencial; limpe com `net use \\<servidor>\C$ /delete /y` e publique de novo; (b) *"Nome de usuário ou senha incorretos"* — credencial do usuário de IIS **rotacionada** no servidor de PROD (a senha anterior ainda autentica em HML, mas não em PROD); obtenha a senha vigente e passe-a em `-SenhaRemota`.
28. O `web.config` do backend é **regerado a cada publish** (o `deploy.ps1` espelha com `/MIR` excluindo só `appsettings*.json` e `app_offline.htm`). Consequência: com o módulo WhatsApp ativo, o bloco `<environmentVariables>` (`WhatsApp__*`) precisa ser **reaplicado no servidor depois do publish** e o backend **reciclado**. O `smoke.ps1` detecta o **erro de schema** do bloco (falha no check **“Swagger/API”**, com `500` de corpo vazio em todas as rotas) — mas **não** detecta o bloco perdido: esse caso só aparece no módulo (webhook em `503`, envio em `502`) e precisa de `GET /api/v1/whatsapp/saude`. Detalhe em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §7 e [`09-TROUBLESHOOTING.md`](../troubleshooting.md) §13.1.
29. A porta do gateway OpenWA publicada por **Docker** **ignora** o `ufw` da VM (o DNAT do Docker acontece antes da `INPUT` do `ufw`; o tráfego de container passa pela `FORWARD`). A restrição é feita na cadeia **`DOCKER-USER`**, com as regras persistidas por unidade systemd — o Docker recria a cadeia no boot. Detalhe em [`instalacao-openwa.md`](../instalacao-openwa/INDEX.md) §4 e [`09-TROUBLESHOOTING.md`](../troubleshooting.md) §19.7.
30. O mascote 3D do chat (fases 1–4 do plano [`planos/jota-3d/00-PLANO-E-ESFORCO.md`](../../planos/jota-3d/00-PLANO-E-ESFORCO.md)) está no código com o gate `JOTA_3D_HABILITADO = false` (`shared/components/jota-3d/jota-3d.types.ts`): o comportamento publicado é **2D** (poster `assets/images/jota.png`) e nenhum chunk de `three` é baixado. A **preparação da fase 5 está pronta**: os 6 recortes de pose (insumo do image-to-3D e pose-chave de cada clipe) estão em [`planos/jota-3d/referencias/`](../../planos/jota-3d/referencias/) e o roteiro de geração/validação do modelo é [`planos/jota-3d/01-FASE-5-GERACAO-DO-MODELO.md`](../../planos/jota-3d/01-FASE-5-GERACAO-DO-MODELO.md) — o **`.glb` em si continua pendente** (produção externa). Antes de ligar a fase 6, confirme (a) o GLB em `frontend/src/assets/models/jota/jota.glb` (fase 5, ainda pendente — gere pelo roteiro acima), (b) o MIME `.glb`/`.gltf`/`.bin`/`.ktx2` no `web.config` publicado e (c) o próprio poster — no working tree desta revisão `frontend/src/assets/images/jota.png` não está presente (existe só `assets/images/JOTA/JOTA-1.png`). `GET /assets/models/jota/jota.glb` deve responder `200` no IIS antes de qualquer ativação.
31. **Publish de árvore não commitada não é reproduzível.** Em 2026-10-27 HML e PROD receberam o pacote da árvore de trabalho sobre o HEAD `b52d7bc` (sem commit): `BUILD_INFO` aponta para o HEAD, mas o binário publicado contém mudanças não commitadas. O preflight passou porque, na época, só verificava `CARGO_ID` (hoje também cobre `CARGO_Nivel`/`CARGO_FuncaoId`), e o login de PROD caiu em 500 até aplicar os dois scripts de permissões. Trate isso como emergência registrada: o rollback para a tag/commit anterior não reproduz a árvore publicada, e o próximo deploy precisa sair de commit limpo.
32. **Tipo de coluna não entra no gate.** `validate` compila e o preflight confere existência de tabelas/colunas — nenhum dos dois compara o tipo do SQL com o CLR. Armadilha real: `tbcargo.CARGO_Nivel` é `tinyint` e estava mapeada como `int`; o EF lê com `SqlDataReader.GetInt32`, que não converte `byte` para `int`, e o login de usuário **válido** caiu em 500 (`InvalidCastException`), enquanto o fictício seguia em 401. O mapeamento CLR deve **espelhar** o SQL (`byte` ↔ `tinyint`, `short?` ↔ `smallint` etc.). Corrigido em `059183d`.

