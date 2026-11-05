---
id: instalacao-openwa.indice
titulo: "Instalação do gateway OpenWA na VM Ubuntu"
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
tokens_estimados: 300
ficha: docs/modulos/whatsapp.md
---

# Instalação do gateway OpenWA na VM Ubuntu

> Índice do documento dividido. Documento original: [instalacao-openwa.md](../instalacao-openwa.md).
> Ficha: [Atendimento WhatsApp](../../modulos/whatsapp.md).

> Runbook do **OpenWA v0.25.0** instalado na VM `192.168.2.253` (Ubuntu 24.04), **fora** do servidor IIS. É um artefato **complementar**: não substitui a documentação temática (a documentação única do módulo está em [`material-de-projeto/atendimento-whatsapp/README.md`](../../../material-de-projeto/atendimento-whatsapp/README.md)).
>
> **Estado real (2026-10-27): gateway no ar, sem sessões.** O piloto rodou na homologação com o número `central-wa-7` (status `ready`, telefone `558592644863`) e a **produção (`.254`) foi publicada em 2026-10-26** — no máximo **um** backend ativo por sessão (§5). O **reset de dados de 2026-10-27** removeu a sessão `central-wa-2` (`204`) e deixou `GET /api/sessions` em `[]`: **reconecte um número antes da validação manual** (o cadastro em `/whatsapp/numeros` cria, inicia e registra o webhook).
>
> **Segredos nunca entram neste arquivo.** Chaves, senhas, QR Codes e connection strings com credencial são gerados e guardados **na própria VM**. Onde o texto precisa mostrar a forma, ele usa placeholder (`SUA_CHAVE`).

## Partes

- [1. Onde o gateway roda](./01-1-onde-o-gateway-roda.md)
- [2. Pré-requisitos e estado real da VM](./02-2-pre-requisitos-e-estado-real-da-vm.md)
- [3. Instalação (caminho Docker — executado)](./03-3-instalacao-caminho-docker-executado.md)
- [4. Rede e firewall — a parte que engana](./04-4-rede-e-firewall-a-parte-que-engana.md)
- [5. Webhook: um gateway, um webhook por sessão](./05-5-webhook-um-gateway-um-webhook-por-sessao.md)
- [6. Schema no SQL Server](./06-6-schema-no-sql-server.md)
- [7. Configurar o backend no IIS (`web.config`)](./07-7-configurar-o-backend-no-iis-web-config.md)
- [8. Variáveis do backend (Central)](./08-8-variaveis-do-backend-central.md)
- [9. Prova técnica — o que foi verificado, o que falta e o ciclo de vida da sessão](./09-9-prova-tecnica-o-que-foi-verificado-o-que-falta-e-o-ciclo-d.md)
- [10. Backup, restauração e atualização](./10-10-backup-restauracao-e-atualizacao.md)
- [11. Riscos](./11-11-riscos.md)
- [12. Referências](./12-12-referencias.md)
