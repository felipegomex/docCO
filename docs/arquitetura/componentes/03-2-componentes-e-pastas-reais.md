---
id: componentes.03
titulo: "2. Componentes e pastas reais"
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
tokens_estimados: 479
ficha: docs/modulos/README.md
---
> Parte de [02 — Arquitetura vigente](./INDEX.md). Ficha: [Fichas de módulo](../../modulos/README.md).

## 2. Componentes e pastas reais

```text
frontend/
├── angular.json
├── package.json
├── public/
│   └── web.config
└── src/
    ├── main.ts
    ├── environments/
    │   ├── environment.ts
    │   └── environment.prod.ts
    └── app/
        ├── app.config.ts
        ├── app.routes.ts
        ├── features.routes.ts
        ├── core/
        ├── features/
        ├── layout/
        └── shared/

backend/Central_BackEnd/
├── Program.cs
├── Central_BackEnd.csproj
├── Controllers/
├── Services/
├── Models/
├── Data/
├── Dtos/
├── Migrations/
├── Exceptions/
├── Properties/
└── appsettings*.json
```

Papéis dos diretórios:

- `frontend/src/app/features`: telas e serviços de domínio.
- `frontend/src/app/core`: autenticação, guards, interceptor, storage de token e serviços globais.
- `frontend/src/app/layout`: shell autenticado principal.
- `frontend/src/app/shared`: componentes e configuração transversal — inclui `shared/components/jota-3d/`, o mascote do chat (`app-jota-3d`: componente, tipos, loader three.js e specs), com gate `JOTA_3D_HABILITADO = false` e `three.js` só por chunk lazy.
- `backend/Central_BackEnd/Controllers`: contrato HTTP e autorização por endpoint.
- `backend/Central_BackEnd/Hubs`: hubs SignalR — hoje `WhatsAppHub` (`/hubs/whatsapp`), canal de notificação de saída do módulo WhatsApp.
- `backend/Central_BackEnd/Services`: regras de negócio e integrações.
- `backend/Central_BackEnd/Models` e `Data`: modelo EF Core e tabelas.
- `backend/Central_BackEnd/Dtos`: contratos de entrada e saída.
- `backend/Central_BackEnd/Migrations`: evolução do modelo SQL.

