---
id: 06-inventario-de-entidades-e-tabelas.09
titulo: "Cifra das senhas — `ICifrador` e como religar"
tipo: referencia
area: dados
sistema: central
tags: [schema, tabelas, entidades, sqlserver]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Models
  - backend/Central_BackEnd/Data/AppDbContext.cs
  - backend/Central_BackEnd/Migrations
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1175
ficha: docs/modulos/acessos.md
---
> Parte de [Inventário de entidades e tabelas](./INDEX.md). Ficha: [Acessos Empresas](../../../modulos/acessos.md).

### Cifra das senhas — `ICifrador` e como religar

`Services/Seguros/Cifrador.cs` define `ICifrador` (`Cifrar`, `Decifrar`, `Implementacao`, `EhCifradoDeVerdade`) com duas implementações:

| Implementação | Estado | Comportamento |
|---|---|---|
| `CifradorEmBranco` | fallback de teste, **NÃO registrado** | Grava e devolve texto puro. Emite `LogWarning` **uma vez por processo** (`Interlocked.Exchange` em `_jaAvisou`). `Implementacao = "em branco (texto puro)"`, `EhCifradoDeVerdade = false`. |
| `CifradorAesGcm` | **ATIVA** (registrada em `Program.cs`) | AES-256-GCM, IV aleatório de 12 bytes **por registro**, tag de 16 bytes. Envelope gravado: `v1:{b64 iv}:{b64 tag}:{b64 ciphertext}`. Chave base64 de 32 bytes em `Acessos:ChaveCifra`. **Valida a chave no construtor e lança**, nesta ordem: (a) ausente; (b) **marcador de placeholder** — `TROQUE_POR_UMA_CHAVE`, `DEV_CHAVE_NAO_USAR`, `CHANGEME`, `CHANGE_ME` (comparação `OrdinalIgnoreCase` por substring), com mensagem que aponta `GerarChaveBase64()`; (c) base64 inválido; (d) tamanho ≠ 32 bytes. A checagem de (b) vem **antes** do `Convert.FromBase64String` de propósito: o placeholder versionado contém `_`, logo não é base64 e sem ela o erro seria "não é um base64 válido", que não diz o que fazer. `GerarChaveBase64()` (estático) gera a chave do servidor com `RandomNumberGenerator`, 32 bytes já em base64. O construtor que lê `IConfiguration` emite `LogInformation` no boot lembrar do backfill. `Decifrar` de um valor **sem** o prefixo `v1:` devolve o texto como está, para as linhas gravadas antes da cifra continuarem legíveis. |

As colunas `RDP_/VPN_/ADK_/BCO_SenhaCifrada` são `nvarchar(400)` **já dimensionadas** para o envelope. Por isso **não há `ALTER TABLE`** ao religar — este é o motivo de a coluna ter sido criada grande desde o início.

**Procedimento de liberação (4 passos, nesta ordem):**

1. **Gerar a chave do servidor** — `CifradorAesGcm.GerarChaveBase64()` ou, em PowerShell, `[Convert]::ToBase64String([Security.Cryptography.RandomNumberGenerator]::GetBytes(32))`. Nunca no repositório.
2. **Gravar a chave** no `appsettings.json` do IIS **ou** na variável de ambiente `Acessos__ChaveCifra`, sob canal seguro.
3. **Confirmar o registro no DI** em `Program.cs`: a implementação ativa é `AddSingleton<ICifrador, CifradorAesGcm>();`. `CifradorEmBranco` não está registrado.
4. **Fazer o backfill** das linhas existentes: para cada `*_SenhaCifrada` sem o prefixo `v1:`, aplicar `Cifrar` uma vez.

⚠️ **Risco: criptografia ligada sem backfill = aparência de segurança sem segurança.** Sem o passo 4, as linhas antigas continuam em texto puro e `Decifrar` cai no caminho de compatibilidade (ausência do prefixo `v1:`) e devolve o texto puro **silenciosamente** — a tela para de avisar e o `ehCifradoDeVerdade` passa a ser `true`, dando a impressão de que tudo está cifrado quando metade do histórico não está. O passo 4 é o único que fecha a lacuna, e por isso não pode ser pulado nem adiado para "depois".

`GET /api/v1/acessos/estado-cifra` mostra a implementação registrada (`ehCifradoDeVerdade = true` com AES), mas **não** mede o backfill: a confirmação real é contar as linhas `*_SenhaCifrada` sem o prefixo `v1:` e verificar que o total é zero.

**Rotação de chave (não suportada automaticamente).** Para trocar a chave: gerar a nova no servidor, re-cifrar cada `*_SenhaCifrada` (ler com a antiga → `Cifrar` com a nova — não há script versionado para isso), atualizar a configuração, reiniciar e validar com `estado-cifra`. Exige janela de manutenção.

**Chave de desenvolvimento.** `appsettings.Development.json` é **ignorado pelo git** ([backend/.gitignore](../../../../backend/.gitignore), linha 6), então a chave usada em dev **nunca foi versionada**. Já `appsettings.json` **é** versionado e mantém o placeholder de propósito: esse é o desenho correto — uma chave real no repositório é que seria o problema, e `CifradorAesGcm` rejeita o placeholder com mensagem acionável.

⚠️ **A cifra não cobre `tboperador`.** `tboperador.SENHA` é `varchar(10)` e guarda material equivalente à senha **em claro** — não é hash protetor: o comparador `SegurancaHelper.SenhasIguais` hasheia os dois lados, então o valor gravado autentica e a leitura da linha equivale a comprometimento. É risco de segurança conhecido; migrar o login é projeto separado (ver [`02-ARQUITETURA.md`](../../componentes/INDEX.md)).

