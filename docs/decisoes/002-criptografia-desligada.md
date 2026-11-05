---
titulo: "ADR 002 — Criptografia do Acessos Empresas desligada (`CifradorEmBranco`)"
status: "substituido"
data: "2026-10-18"
---

> Substituído pelo [ADR 005](./005-cifra-aes-ativa.md) em 2026-10-28: o DI
> registra `CifradorAesGcm`, não `CifradorEmBranco`.
>
> Data de registro. A data da decisão não está documentada nas fontes deste ADR.

# ADR 002 — Criptografia do Acessos Empresas desligada (`CifradorEmBranco`)

## Contexto

As senhas dos 5 tipos de acesso do módulo Acessos Empresas são gravadas nas colunas `RDP_/VPN_/ADK_/BCO_SenhaCifrada` (`nvarchar(400)`). `Services/Seguros/Cifrador.cs` define a abstração `ICifrador` (`Cifrar`, `Decifrar`, `Implementacao`, `EhCifradoDeVerdade`) com duas implementações: `CifradorEmBranco` e `CifradorAesGcm` (AES-256-GCM, envelope `v1:{b64 iv}:{b64 tag}:{b64 ciphertext}`, chave base64 de 32 bytes em `Acessos:ChaveCifra`, recusa de placeholders — `TROQUE_POR_UMA_CHAVE`, `DEV_CHAVE_NAO_USAR`, `CHANGEME`, `CHANGE_ME` — antes do decode base64).

## Decisão

Manter a criptografia **desligada por decisão deliberada**: a implementação **ativa** registrada em `Program.cs` é `CifradorEmBranco`, que grava e devolve texto puro e emite um `LogWarning` **uma vez por processo**. `CifradorAesGcm` está pronta, mas **não está registrada no DI** — é desligamento deliberado, não item ausente. O estado real é visível em `GET /api/v1/acessos/estado-cifra` (`implementacao`, `ehCifradoDeVerdade`, `eventosAuditoriaPerdidos`) e a tela exibe o aviso de texto puro. Mitigações em vigor enquanto a cifra não é liberada: colunas dimensionadas para o envelope (ligar a cripto depois **não** exige `ALTER TABLE`), abstração `ICifrador` no DI, aviso no log e na interface, e auditoria completa (`tbauditoriaacesso` nunca grava a senha). A liberação, quando decidida, segue o procedimento de 4 passos documentado — gerar a chave **no servidor** (nunca no repositório), gravá-la em `appsettings` do IIS ou variável `Acessos__ChaveCifra`, trocar o registro no DI e **fazer o backfill** das linhas sem prefixo `v1:`. A cifra **não cobre** `tboperador.SENHA` (`varchar(10)` em texto claro, comparação SHA-256 estática) — migrar o login é projeto separado.

## Consequências

- As credenciais ficam **em texto puro** até a liberação; `appsettings.Development.json` é ignorado pelo git (chave de dev nunca versionada) e `appsettings.json`, versionado, mantém o placeholder de propósito.
- **Ligar a cifra sem backfill = aparência de segurança sem segurança**: `Decifrar` cai no caminho de compatibilidade (valor sem `v1:` devolve o texto puro silenciosamente), a tela para de avisar e `ehCifradoDeVerdade` vira `true` mascarando metade do histórico em claro. O backfill não é opcional nem adiável.
- Rotação de chave não é suportada automaticamente: troca de chave exige re-cifração manual (ler com a antiga → `Cifrar` com a nova), janela de manutenção e validação por `estado-cifra`.

## Fonte

- `backend/Central_BackEnd/Services/Seguros/Cifrador.cs` (`ICifrador`, `CifradorEmBranco`, `CifradorAesGcm`, `GerarChaveBase64`)
- `docs/arquitetura/banco-dados.md` — seção "Cifra das senhas — `ICifrador` e como religar"
- `docs/referencia/regras-negocio.md` — seção "Dados sensíveis" e "Planejados/recomendados"
- `docs/operacao/troubleshooting.md` — seção 10.2 (senhas, cifra e revelação)
