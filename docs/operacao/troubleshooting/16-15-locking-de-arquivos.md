---
id: sec.16-15-locking-de-arquivos
titulo: "15. Locking de arquivos"
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
tokens_estimados: 530
ficha: docs/modulos/README.md
---
> Ficha: [Fichas de módulo](../../modulos/README.md).

> Parte de [docs/operacao/troubleshooting.md](../troubleshooting.md).

# 15. Locking de arquivos

### Lock local

| SituaÃ§Ã£o | Causa | SoluÃ§Ã£o |
|---|---|---|
| build falha ao substituir DLL/JS | `dotnet run`, `ng serve` ou build anterior ainda aberto | encerre apenas processos cujo `CommandLine` aponte para este repositÃ³rio |
| `node_modules` bloqueado no Windows | processo Node/editor/antivÃ­rus | feche o frontend, encerre o processo identificado e repita `npm ci` |
| `publish_output` bloqueado | backend em execuÃ§Ã£o ou scanner | pare o backend local; aguarde o scanner; nÃ£o apague a fonte |
| deploy incremental mantÃ©m arquivo obsoleto | `/E` nÃ£o apaga saÃ­das anteriores | com processos fechados, remova somente `dist`, `publish_output` e/ou `deploy` gerados e refaÃ§a build/package |

Para localizar sem matar indiscriminadamente processos:

```powershell
Get-CimInstance Win32_Process |
  Where-Object { $_.Name -in @('dotnet.exe', 'node.exe') } |
  Select-Object ProcessId, Name, CommandLine
```

### Lock remoto da DLL

O script cria `app_offline.htm` e espera 3 segundos por padrÃ£o. O worker do IIS pode continuar segurando a DLL. `robocopy` cÃ³digo 11 Ã© falha; o script trata qualquer cÃ³digo 8 ou maior como erro.

Procedimento:

1. Pare a tentativa de deploy; nÃ£o remova `app_offline.htm`.
2. Aguarde 1â€“2 minutos pela reciclagem do worker.
3. Confirme que o pacote local ainda Ã© o do `BUILD_INFO` esperado.
4. Repita a publicaÃ§Ã£o sem rebuild e com backup, usando a mesma credencial segura.
5. Se a cÃ³pia concluir, confirme que `app_offline.htm` foi removido e recicle o pool do ambiente se necessÃ¡rio.
6. Rode o smoke e o QA funcional.

Evite encerrar `w3wp.exe` manualmente. O `app_offline.htm` e a reciclagem do Application Pool sÃ£o o mecanismo controlado.

