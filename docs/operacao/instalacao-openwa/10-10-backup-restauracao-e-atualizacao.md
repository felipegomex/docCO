---
id: instalacao-openwa.10
titulo: "10. Backup, restauração e atualização"
tipo: operacao
area: operacao
sistema: central
tags: [openwa, gateway, whatsapp, docker]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - backend/Central_BackEnd/Services/WhatsApp
  - scripts
testes:
  - docs/
tokens_estimados: 675
ficha: docs/modulos/whatsapp.md
---
> Parte de [Instalação do gateway OpenWA na VM Ubuntu](./INDEX.md). Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

## 10. Backup, restauração e atualização

### 10.1 Backup do volume (o que mantém o número pareado)

O diretório de dados guarda o **perfil de sessão** (o que faz o número voltar conectado sem novo QR), o SQLite do gateway, as mídias, a configuração gerada e a chave do primeiro boot. No caminho **Docker**, o volume nomeado **`openwa_openwa-data`** é o que precisa ser copiado — **é ele que mantém o número pareado**. Os scripts `scripts/backup.sh` e `scripts/restore.sh` do projeto rodam **dentro do container**, onde o volume está montado.

Backup do volume (na VM, com o container **parado** ou com a sessão parada):

```bash
cd ~/openwa
docker compose stop openwa-api          # a sessão não pode estar autenticando durante a cópia
docker run --rm -v openwa_openwa-data:/data -v "$HOME/backups:/backup" \
  alpine tar czf /backup/openwa-data-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .
docker compose start openwa-api
```

Regras:

- **Pare as sessões antes do backup** (`POST /api/sessions/{id}/stop`): o estado de autenticação do navegador é copiado ao vivo, e uma cópia feita com a sessão ativa pode exigir re-pareamento. Reinicie as sessões depois (`POST /api/sessions/{id}/start`).
- **Copie o diretório inteiro**: a sessão só volta sem QR se os arquivos de perfil tiverem vindo junto.
- ⚠️ **O backup contém chaves de API, credenciais e o estado de autenticação do WhatsApp.** Cifre em repouso, restrinja o acesso e **nunca** anexe a um ticket de suporte.

### 10.2 Atualização de versão

1. **Pare as sessões** e faça o backup (§10.1) — o backup precisa ser **anterior** ao primeiro start na nova versão.
2. Troque a tag: `git fetch --tags && git checkout v<nova-versão>` e **atualize a `image:` do override** para a nova versão (a tag do repositório e a tag da imagem andam juntas).
3. Suba, valide a saúde e rode as migrations do gateway.
4. **Restaure o `sessions/` do backup pré-atualização** se a atualização **cruzou uma versão maior do Chrome**. O perfil do navegador **não é portável entre versões maiores do Chrome**: um Chrome mais antigo apaga silenciosamente o IndexedDB que um Chrome mais novo abriu — e é ali que o `whatsapp-web.js` guarda o login do WhatsApp. O sintoma é característico: toda sessão já pareada volta a pedir QR, sem causa aparente no log.
5. Na volta (rollback), o mesmo se aplica: restaurar `sessions/` do backup **pré-atualização**.

