---
id: 05-4-roteamento.09
titulo: "4.7 Rotas de Ideias, Melhorias e Erros"
tipo: referencia
area: frontend
sistema: central
tags: [frontend, angular, rotas]
status: ativo
revisado_em: "2026-10-29"
fonte:
  - frontend/src/app
testes:
  - frontend/src/app
tokens_estimados: 560
ficha: docs/modulos/ideias.md
---
> Parte de [4. Roteamento](./INDEX.md). Ficha: [Ideias, Melhorias e Erros](../../../modulos/ideias.md).

### 4.7 Rotas de Ideias, Melhorias e Erros

`ideias.routes.ts` é lazy e filho de `path: "ideias"` em `features.routes.ts`. Herdam `authGuard` do shell em `/`.

| URL | Componente |
|---|---|
| `/ideias` | redirect para `lista` (`pathMatch: full`) |
| `/ideias/lista` | `ListaIdeiasComponent` |
| `/ideias/nova` | `NovaIdeiaComponent` (seletor de tipo aberto) |
| `/ideias/nova/:tipo` | `NovaIdeiaComponent` com o tipo já escolhido |
| `/ideias/:id` | `DetalheIdeiaComponent` |
| `/ideias/**` | redirect para `lista` |

**Guard de entrada (any-of).** A rota `ideias` em `features.routes.ts` usa `permissaoGuard([ 'aba.sistema.ideias', 'aba.ciaa.ideias', 'aba.implantacao.ideias' ])`: o mesmo `/ideias/lista` é a porta do Sistema, do CIAA e da Implantação. Quem não tem nenhum dos três recursos é barrado na navegação; o backend continua sendo a barreira final e devolve `404` — que cobre tanto "não existe" quanto "existe e não é seu" —, e a tela traduz isso em "Solicitação não encontrada".

⚠️ **O mapa de suporte de operação é obrigatório neste módulo.** `IdeiaDetalhe.OperacoesPermitidas` devolve as **13** operações da matriz de `IdeiasPermissaoService`, mas a API pública atende **7**. `Aprovar`, `VincularKanban`, `AnalisarIa` e `Especificar` são liberadas sem endpoint (fases F6, F7 e F14 não implementadas). Iterar essa lista direto renderizaria quatro botões que o servidor rejeita com `404`.

A solução está em `ideias.constants.ts`: `OPERACOES_SUPORTADAS` mapeia cada operação para `{ chave, rotulo, icone, endpoint, mostrarDesabilitada, motivoIndisponivel }`, e `resolverAcoes(permitidas)` devolve duas listas — `disponiveis` (clicáveis) e `bloqueadas` (sem endpoint). As bloqueadas entram desabilitadas, com o motivo no `title`, porque somi-las da barra deixaria o gestor sem explicação para a ausência. **Qualquer operação nova que o backend adicione precisa entrar nesse mapa**, ou o botão não aparece.

