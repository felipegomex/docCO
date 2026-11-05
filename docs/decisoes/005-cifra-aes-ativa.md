---
titulo: "ADR 005 — Cifra AES-256-GCM ativa no Acessos Empresas"
status: "aceito"
data: "2026-10-28"
---

# ADR 005 — Cifra AES-256-GCM ativa no Acessos Empresas

## Contexto

O [ADR 002](./002-criptografia-desligada.md) registrou a decisão de manter a
criptografia **desligada**, com `CifradorEmBranco` registrado no DI. Desde
então o código mudou: `Program.cs` passou a registrar `CifradorAesGcm`
(AES-256-GCM, envelope `v1:{b64 iv}:{b64 tag}:{b64 ciphertext}`) como
`ICifrador`.

## Decisão

Manter `CifradorAesGcm` como implementação ativa do `ICifrador`.

- `CifradorEmBranco` permanece no código apenas como implementação de
  teste/fallback e **não** está registrada no DI.
- `CifradorAesGcm.Decifrar` mantém compatibilidade: valor sem prefixo `v1:` é
  devolvido como texto puro. Isso permite ler linhas antigas durante a
  transição, mas **não prova** que o backfill terminou.
- `GET /api/v1/acessos/estado-cifra` informa a implementação registrada e
  `ehCifradoDeVerdade`; ele **não** mede a cobertura do backfill. A conferência
  de cobertura precisa contar as linhas sem prefixo `v1:`.
- A chave continua fora do repositório: `Acessos:ChaveCifra` (user-secrets em
  desenvolvimento ou `Acessos__ChaveCifra` no servidor).

## Consequências

- Escritas novas são cifradas; leituras antigas continuam funcionando pelo
  caminho de compatibilidade.
- O aviso de "texto puro" da tela deixa de ser verdadeiro para linhas novas,
  mas pode continuar verdadeiro para linhas ainda não migradas.
- `tboperador.SENHA` (`varchar(10)`, SHA-256 de comparação) continua **fora**
  do alcance desta cifra — é projeto separado.

## Fonte

- `backend/Central_BackEnd/Program.cs` — registro de `ICifrador`.
- `backend/Central_BackEnd/Services/Seguros/Cifrador.cs` — `CifradorEmBranco`,
  `CifradorAesGcm` e compatibilidade de `Decifrar`.
- `backend/Central_BackEnd/Controllers/AcessosController.cs` — `estado-cifra`.
- [ADR 002](./002-criptografia-desligada.md) — decisão anterior, substituída.
