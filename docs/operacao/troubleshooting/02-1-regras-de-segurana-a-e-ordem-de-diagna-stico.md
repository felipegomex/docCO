---
id: sec.02-1-regras-de-segurana-a-e-ordem-de-diagna-stico
titulo: "1. Regras de seguranÃ§a e ordem de diagnÃ³stico"
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
tokens_estimados: 451
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 1. Regras de seguranÃ§a e ordem de diagnÃ³stico

- NÃ£o cole senha, chave JWT, string de conexÃ£o completa, cookie ou token em issue, terminal compartilhado, log ou documentaÃ§Ã£o.
- Use `user-secrets` no desenvolvimento e armazenamento seguro/variÃ¡veis de ambiente no IIS.
- Ao diagnosticar SQL, Database Explorer e IIS, mantenha as credenciais fora da linha de comando e das capturas de tela.
- NÃ£o mate indiscriminadamente todos os processos `dotnet` ou `node`; outras aplicaÃ§Ãµes podem estar usando as mesmas portas.
- Antes de alterar banco ou servidor, faÃ§a backup e confirme o ambiente.

> **Credencial do servidor â€” fato atual:** `scripts/deploy/deploy.ps1` **nÃ£o** contÃ©m credencial fixa nem fallback (a senha fixa anterior foi removida por exposiÃ§Ã£o no histÃ³rico do Git). A publicaÃ§Ã£o **exige** `-SenhaRemota` (`SecureString`) explÃ­cito e aborta sem ele. Execute sempre com `Get-Credential` ou `SecureString` de secret manager; nunca informe a senha como texto.

Ordem mÃ­nima:

1. Reproduza na mesma URL e no mesmo navegador.
2. Execute o gate sem pular nenhum projeto:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\validate.ps1
```

3. Separe o problema em build, processo, rede, autenticaÃ§Ã£o, banco ou conteÃºdo publicado.
4. Consulte o terminal que iniciou cada processo e os logs do IIS.
5. SÃ³ altere cÃ³digo ou infraestrutura depois de identificar a causa.

