---
id: componentes.09
titulo: "8. CORS, IIS e publicação"
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
tokens_estimados: 440
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 8. CORS, IIS e publicação

### CORS

A policy `Angular` permite explicitamente:

- `http://192.168.2.130:1010`;
- `http://localhost:4200`;
- `http://localhost:1010`.

Ela permite qualquer header e método, mas habilita `AllowCredentials`. Não há wildcard de origem. Alterações de origem exigem atualizar a policy e o contrato de deploy.

### IIS

- O backend usa `http://0.0.0.0:1009` no perfil de execução e expõe Swagger sob `/swagger` quando `SwaggerEnabled` está habilitado.
- O frontend de desenvolvimento usa `http://localhost:4200`.
- O frontend de produção é publicado em `Suporte_Front` na porta `1010`.
- `frontend/public/web.config` requer URL Rewrite e reescreve URLs Angular para `/index.html` quando não há arquivo ou diretório. Ele também declara `<staticContent>` com `mimeMap` para `.glb`, `.gltf`, `.bin` e `.ktx2` (modelos do mascote JOTA 3D): sem esse bloco o IIS devolve `404.3` para `/assets/models/*`.
- O deploy copia `browser/` para o site IIS; não há processo Node no site.
- O backend é publicado em `Suporte_Back` na porta `1009`; `app_offline.htm` é usado durante a substituição dos arquivos.
- O mirror de publicação exclui `appsettings*.json`, preservando a configuração do servidor.

HSTS é habilitado fora de Development, mas as URLs de frontend, API e cookie registradas são HTTP. A coexistência entre HSTS e HTTP deve ser tratada como limitação de implantação, não como garantia de transporte seguro.

