---
id: sec.14-9-acessos-empresas
titulo: "9. Acessos Empresas"
tipo: referencia
area: qa
sistema: central
tags: [qa, historias, testes]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
  - backend/Central_BackEnd
testes:
  - backend/Central_BackEnd.Tests
tokens_estimados: 1224
ficha: docs/modulos/acessos.md
---
> Ficha: [Acessos Empresas](../../modulos/acessos.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 9. Acessos Empresas

### CF-12 — Abrir o detalhe da unidade, cruzar o gate e revelar credenciais

- **Objetivo:** Ver os 5 tipos de acesso de uma unidade, mantendo o gate de senha do operador e deixando rastro de cada revelação.
- **Pré-condições:** operador autenticado; unidade com ao menos um acesso cadastrado; credencial do operador disponível para o teste, **sem inseri-la neste documento**.
- **Passos:** 1. Abrir `/acessos/clientes/:id` (ou pelo menu ⋮ da lista) — a senha foi paga **na listagem**, que emite o ticket de `VISUALIZAR` (2 min, ou 5 min se marcado "Lembrar por 5 minutos"; reusável na mesma empresa), então o detalhe abre direto. 2. Navegar pelas abas RDP, VPN, AnyDesk, Banco e Actyon Web **sem redigitar a senha**. 3. Na aba RDP/VPN, usar o botão de olho em um acesso e observar a senha. 4. Copiar um valor. 5. Esperar a senha sumir sozinha. 6. Criar/editar um acesso pelo modal — este pede **usuário digitado + senha** de novo, porque cada escrita usa um ticket `EDITAR` de uso único; o gate usa o layout padrão de modal (`modal-header` com título + fechar, corpo com campos e nota de auditoria, rodapé com "Cancelar" e "Confirmar 🔒"). 7. Tentar uma segunda escrita seguida sem redigitar: o gate reabre, porque o ticket anterior foi consumido.
- **Resultado esperado:** o gate **não repete** na troca de abas enquanto o ticket de `VISUALIZAR` estiver vivo (2 min, ou 300 s com "Lembrar", escopado à empresa), mas **repete a cada operação sensível**, já que o ticket de `EDITAR` é de uso único; o modal de gate pede **usuário e senha** no layout padrão Bootstrap (cabeçalho com ícone de cadeado, corpo com campos marcados e rodapé com ações) — usuário divergente do JWT responde 401 auditado **sem** queimar tentativa; credencial errada mantém o formulário com erro e limpa a senha (401/429 traduzidos com a contagem de tentativas). Depois do gate, cada aba lista seus acessos em **cards** (`.acesso-cards`/`.acesso-cardacesso`) com título, pares rotulo/valor, badge status e as ações (revelar, copiar, editar, inativar, "Abrir Actyon Web"), e um resumo (ativos/total). No RDP, **Servidor** e **IP (`ip:porta`)** são campos separados com cópia própria — no detalhe e no modal Visualizar. Inativar (`DELETE /rdp|vpn|anydesk|banco|actyon-web/{id}`) envia o ticket `EDITAR` da **unidade** (`clienteAcessoId`), não o id do acesso. O botão de olho existe **apenas em RDP e VPN**; a senha revelada **se esconde sozinha em 30 segundos**. Toda revelação exibe um toast informando que ficou registrada. O estado da cifra aparece no topo. Salvar ou inativar um acesso recarrega o detalhe e atualiza as flags da aba e do resumo.
- **Estado atual/bloqueio:** ⚠️ as escritas novas são cifradas com AES-256-GCM (`CifradorAesGcm` ativo); linhas antigas sem o prefixo `v1:` continuam legíveis por compatibilidade e podem permanecer em texto puro até o backfill. O aviso de texto puro na tela reflete `ehCifradoDeVerdade`, não a cobertura do backfill. O botão "olho" inexiste em AnyDesk e Banco **por decisão**: o backend não expõe revelação nesses dois tipos. A senha revelada fica no estado do componente por 30 s — em tela compartilhada, esse é o intervalo de exposição. A autorização é apenas `[Authorize]`: **qualquer operador autenticado** cria, altera e inativa; a segregação é por auditoria. O "Lembrar" estende o ticket de `VISUALIZAR` a 5 min naquele cliente; é conveniência do operador, não privilégio novo.
- **Rotas/endpoints/tabelas:** `/acessos/clientes/:id`; `POST /api/v1/acessos/clientes/{id}/autorizar` (gate — corpo `{ usuario, senha, finalidade, lembrar }`, emite o ticket + `validoPor`), `GET /acessos/estado-cifra`, `GET /acessos/clientes/{id}`, `GET /acessos/clientes/{id}/{rdp|vpn|anydesk|banco|actyon-web}`, `GET /acessos/rdp/{id}/senha`, `GET /acessos/vpn/{id}/senha`, `POST|PUT|DELETE /acessos/{rdp|vpn|anydesk|banco|actyon-web}[/{id}]`; header `X-Acesso-Ticket` nas operações sensíveis; `tbclienteacesso`, `tbcliente_grupo` (grupos), `tbacessordp`, `tbacessovpn`, `tbacessoanydesk`, `tbacessobanco`, `tbacessoactyonweb`, `tbauditoriaacesso` (REVEAL_SENHA e as tentativas de senha), `tbauditoriaimplantacao` (CREATE/UPDATE/DELETE), `tboperador` (gate).
- **Status/automação:** **cenário documentado**; não há teste de UI. O backend tem suíte (`backend/Central_BackEnd.Tests`, 93 casos) para validadores, cifra, a regressão de `TemSenha`, o TTL do ticket (incluindo o de 5 min com `lembrar`) e o Kanban Geral — **não** para o fluxo de tela nem para a fila de auditoria.

