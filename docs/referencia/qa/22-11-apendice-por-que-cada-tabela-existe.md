---
id: sec.22-11-apendice-por-que-cada-tabela-existe
titulo: "11. Apêndice — por que cada tabela existe"
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
tokens_estimados: 3189
ficha: docs/modulos/database-explorer.md
---
> Ficha: [Database Explorer](../../modulos/database-explorer.md).

> Parte de [docs/referencia/qa-historias.md](../qa-historias.md).

# 11. Apêndice — por que cada tabela existe

Leitura de negócio das 51 tabelas (detalhe técnico em [`04-ESTRUTURA-DADOS.md`](../../arquitetura/banco-dados.md)). Regra geral: o sistema **cria** o que é operação própria e **só lê** o que nasce no ERP/legado — para não duplicar cadastro.

### Acessos Empresas (7)
| Tabela | Por que foi criada |
|---|---|
| `tbclienteacesso` | É a **unidade/filial** (o cliente atendido): dona de todos os acessos e amarrada a uma empresa do ERP, sem duplicar. |
| `tbacessordp` | Guarda o acesso remoto **RDP** (servidor, usuário, senha) que o suporte usa para entrar na máquina do cliente. |
| `tbacessovpn` | Guarda o acesso **VPN** (tipo, gateway, usuário, senha, arquivo de configuração) para entrar na rede do cliente. |
| `tbacessoanydesk` | Guarda o número **AnyDesk** para assistência remota rápida. |
| `tbacessobanco` | Guarda o acesso ao **banco de dados** do cliente (servidor, versões SQL/COB, usuário, senha, arquivo) para suporte e consultas. |
| `tbacessoactyonweb` | Guarda o endereço web do **Actyon** do cliente (URL, ambiente) — acesso por link, sem senha. |
| `tbauditoriaacesso` | Registra **quem viu ou revelou cada senha, de qual empresa, quando e de onde** — a trilha de segurança do módulo. |

### Implantação (15)
| Tabela | Por que foi criada |
|---|---|
| `tbprojeto` | O **projeto de implantação** (cliente, tipo, responsável, go-live): o contêiner de todo o trabalho. |
| `tbprojetoetapa` + checklist, documento, histórico, comentário (5) | A **jornada fixa de 9 etapas** de cada projeto, com o que falta fazer, os documentos, o que mudou e as conversas. |
| `tbtarefa` + responsável, chamado, apontamento (4) | O **trabalho executável**: cards com prazo e função, quem mais ajuda, qual chamado originou e as **horas apontadas**. |
| `tbcolunakanban` + `tbcolunafuncao` | O **catálogo compartilhado** de colunas do Kanban (ordem, cor, limite) e a **visão de cada setor** sobre ele (quais colunas aparecem e a máscara de nome). |
| `tbfuncaoetapa` | O catálogo de **etapas por função** — o eixo do Kanban Geral. |
| `tbbloqueio` | O **bloqueio como atributo do card** (o que travou, desde quando, histórico). |
| `tbtipoprojeto` | O catálogo de **tipos de projeto** oferecido no cadastro. |

### Agenda (3)
| Tabela | Por que foi criada |
|---|---|
| `tbagenda` | Os **compromissos** (responsável, horário, SLA, recorrência). |
| `tbtipoevento` | Os **tipos de evento** (nome, cor) para classificar a agenda. |
| `tbagendaparticipante` | Quem **participa** de cada evento. |

### Dashboard Comercial/Financeiro (5)
| Tabela | Por que foi criada |
|---|---|
| `tblead` | O **funil comercial**: leads em prospecção, etapa e arquivamento. |
| `tbleadcontato` | O **histórico de contatos** feitos com cada lead. |
| `tboportunidade` | **Uma linha por produto ofertado** (valores, quantidade, motivo da perda). |
| `tborcamento` | Os **orçamentos** financeiros da operação. |
| `tbcusto` | Os **custos mensais** por tipo e competência. |

### Atendimento WhatsApp (7)
| Tabela | Por que foi criada |
|---|---|
| `tbwhatsapp_numero` | O **número conectado** (sessão do gateway, fila, estado e texto de encerramento). |
| `tbwhatsapp_contato` | **Quem está do outro lado** (uma linha por chat). |
| `tbwhatsapp_conversa` | O **canal** da conversa: o par (número, chat) e o atendimento ativo. |
| `tbwhatsapp_atendimento` | O **card do Kanban** (status, responsável, prioridade, marcas de tempo do ciclo). |
| `tbwhatsapp_mensagem` | O **histórico da conversa** (recebida, enviada e nota interna), com o estado de entrega. |
| `tbwhatsapp_evento` | A **trilha de atribuição e ciclo de vida** (append-only): quem assumiu, transferiu, devolveu, resolveu, reabriu. |
| `tbwhatsapp_webhook_fila` | A **fila durável** das entregas do gateway — é o que impede perder mensagem no recycle do pool do IIS e o que faz a dedupe. |

### RBAC (4)
| Tabela | Por que foi criada |
|---|---|
| `tbcargo` | O **cargo** do operador (Suporte N1, Tech Lead, Desenvolvedor, Visualizador…), separado do setor (`FUNCAO_ID`). |
| `tbpermissao_recurso` | O **catálogo de recursos** protegidos (abas, APIs, relatórios, tabelas), com a chave estável usada nas policies. |
| `tbperm_cargo` | A **matriz do cargo**: quais ações cada cargo tem em cada recurso e até onde (Próprio/Função/Tudo). |
| `tbperm_operador` | A **exceção individual**: concede ou revoga ações para um operador; a negativa vence o cargo. |

### Base e trilhas (4)
| Tabela | Por que foi criada |
|---|---|
| `tboperador` | **Quem acessa** o sistema (login, perfil, função). |
| `tbfuncao` | As **funções/setores** (Suporte, Implantação, CIAA…) que organizam o Kanban e o escopo de leitura; o **cargo** RBAC fica em `tbcargo`. |
| `tbrefreshtoken` | Mantém a **sessão** (renovar login sem pedir senha de novo, revogar no logout). |
| `tbauditoriaimplantacao` | Trilha de **quem criou/alterou/apagou** o quê na Implantação (antes/depois). |

### Legadas — só leitura (6, nascem no ERP/legado)
| Tabela | Por que só lê |
|---|---|
| `tbcliente` | **Fonte única de clientes** (fantasia, CNPJ, grupo): o Acessos vincula a unidade a ela em vez de recadastrar. |
| `tbcliente_grupo` | **Grupos econômicos** para filtrar e agrupar (substituiu tabela própria que foi dropada). |
| `tbchamado` | **Chamados do sistema legado**, vinculados às tarefas sem recriá-los. |
| `tbfuncionario` | Vínculo operador–função usado em filtros (não substitui `tboperador`). |
| `tbcontrato` | **Contratos do legado**: base de licenças, valores e reajustes projetados no Dashboard. |
| `tbproduto` | **Catálogo de produtos** da proposta (produtos nascem no legado). |

### WA-30 — Consultar métricas de qualidade (taxa reabertura, FCR, transferências, CSAT)

- **Objetivo:** Visualizar indicadores de qualidade do atendimento na aba "Qualidade" das métricas.
- **Pré-condições:** operador autenticado com papel `Administrador`; atendimentos no período.
- **Passos:** 1. Acessar `/whatsapp/metricas`. 2. Clicar na aba "Qualidade". 3. Ler os indicadores: taxa de reabertura, FCR (First Contact Resolution), total de transferências, CSAT (se disponível).
- **Resultado esperado:** a aba exibe os quatro indicadores agregados no período filtrado. Valores calculados no backend em minutos úteis (descontando espera do cliente).
- **Estado atual/bloqueio:** CSAT pode vir nulo se não houver pesquisa de satisfação implementada.
- **Rotas/endpoints/tabelas:** `/whatsapp/metricas`; `GET /api/v1/whatsapp/metricas/qualidade` (admin); `MetricasQualidadeWhatsApp` com `Qualidade`, `PorCategoria[]`, `PorCliente[]`.
- **Status/automação:** **cenário documentado**.

### WA-32 — Consultar métricas por cliente (volume, tendência)

- **Objetivo:** Visualizar volume de atendimentos e tendência por cliente na aba "Clientes".
- **Pré-condições:** operador autenticado com papel `Administrador`; atendimentos no período.
- **Passos:** 1. Acessar `/whatsapp/metricas`. 2. Clicar na aba "Clientes". 3. Ler a tabela: cliente (nome do contato), quantidade, tendência (ex.: "estável", "subindo", "descendo").
- **Resultado esperado:** lista de clientes com volume de atendimentos e indicador de tendência calculado no backend.
- **Estado atual/bloqueio:** tendência é calculada comparando o período atual com o anterior de mesmo tamanho.
- **Rotas/endpoints/tabelas:** `/whatsapp/metricas`; `GET /api/v1/whatsapp/metricas/qualidade` (admin); `MetricaClienteWhatsApp[]`.
- **Status/automação:** **cenário documentado**.

### WA-33 — Consultar auditoria do canal (filtros usuário/ação/objeto/período)

- **Objetivo:** Permitir ao administrador auditar ações realizadas no canal WhatsApp.
- **Pré-condições:** operador autenticado com papel `Administrador`.
- **Passos:** 1. Acessar `/whatsapp/auditoria` pelo menu SUPORTE → WhatsApp → Auditoria. 2. Preencher filtros: usuário, ação, objeto, período de/até. 3. Clicar em buscar. 4. Paginar resultados.
- **Resultado esperado:** tabela paginada com colunas: ID, usuário, ação, objeto, objetoId, alteração (JSON/texto), data/hora. Filtros funcionam corretamente. Ordenação por data/hora decrescente.
- **Estado atual/bloqueio:** leitura só admin; escrita de auditoria ocorre automaticamente em todos endpoints de escrita do módulo.
- **Rotas/endpoints/tabelas:** `/whatsapp/auditoria`; `GET /api/v1/whatsapp/auditoria` (admin); `tbwhatsapp_auditoria`.
- **Status/automação:** **cenário documentado**.

### WA-34 — Configurar IA do canal (provedor, chave API, modelo, timeout)

- **Objetivo:** Permitir ao administrador configurar a integração com provedor de IA (Gemini).
- **Pré-condições:** operador autenticado com papel `Administrador`; chave da API Gemini disponível (user-secrets/env).
- **Passos:** 1. Acessar `/whatsapp/ia-configuracao` pelo menu SUPORTE → WhatsApp → Configuração IA. 2. Selecionar provedor (hoje só "Gemini"). 3. Informar chave da API (campo password, mascarado). 4. Informar modelo (ex.: `gemini-3.5-flash`). 5. Definir timeout (segundos, padrão 60). 6. Marcar "Ativo". 7. Salvar.
- **Resultado esperado:** configuração salva; GET subsequente devolve a chave mascarada; chave real armazenada em user-secrets (dev) ou variável de ambiente (prod); auditoria registrada.
- **Estado atual/bloqueio:** provedor único (Gemini); fallback "em breve" se não configurado.
- **Rotas/endpoints/tabelas:** `/whatsapp/ia-configuracao`; `GET/PUT /api/v1/whatsapp/ia/configuracao` (admin); `tbwhatsapp_ia_config`.
- **Status/automação:** **cenário documentado**.

### WA-35 — Usar Assistente IA real (resumo, classificação, sugestões)

- **Objetivo:** Obter assistência de IA para um atendimento específico (resumo, classificação, sugestões de resposta).
- **Pré-condições:** operador autenticado; IA configurada e ativa (`/whatsapp/ia-configuracao`); atendimento existente.
- **Passos:** 1. Abrir `/whatsapp/atendimento/:id`. 2. Clicar na aba lateral "Assistente IA". 3. Aguardar carregamento (chamada ao endpoint). 4. Ler: resumo da conversa, classificação (categoria, prioridade, sentimento, intenção), sugestões (respostas, artigos, procedimentos, próximos passos).
- **Resultado esperado:** painel exibe dados reais vindos do `GET /api/v1/whatsapp/ia/atendimentos/{id}/assistente-ia` (integração com `GeminiAnaliseProvedor`). Se IA não configurada, exibe estado "em breve" com botões desabilitados. **Nenhuma recursão/stack overflow** ao abrir a aba; URL correta `/whatsapp/ia/atendimentos/{id}/assistente-ia`.
- **Estado atual/bloqueio:** integração com provedor existente; falha do provedor → 502/504; não persiste resultado.
- **Rotas/endpoints/tabelas:** `/whatsapp/atendimento/:id`; `GET /api/v1/whatsapp/ia/atendimentos/{id}/assistente-ia`; `AssistenteIaResumoWhatsApp`.
- **Status/automação:** **cenário documentado**.

### WA-36 — Kanban com 4 colunas e filtro "incluir arquivados"

- **Objetivo:** Validar o board com 4 colunas por padrão e a revelação de Finalizado/Arquivado/Visto pelo filtro.
- **Pré-condições:** operador autenticado; atendimentos nos status operacionais, finalizados e vistos.
- **Passos:** 1. Abrir `/whatsapp`. 2. Conferir 4 colunas (Não atendido, Em atendimento, Aguardando cliente, Resolvido). 3. Marcar "incluir arquivados". 4. Conferir as colunas extras (Finalizado, Arquivado e **Visto**) e o grid adaptado.
- **Resultado esperado:** 4 colunas por padrão; filtro revela Finalizado/Arquivado/**Visto**; o filtro de status também só oferece essas três com "incluir arquivados" ligado; contador soma apenas colunas visíveis.
- **Rotas/endpoints/tabelas:** `/whatsapp`; `GET /api/v1/whatsapp/atendimentos` com `incluirArquivados`.
- **Status/automação:** **cenário documentado**.

### WA-37 — Auditoria com gravação real

- **Objetivo:** Validar que as escritas do módulo gravam em `tbwhatsapp_auditoria` e a listagem paginada funciona.
- **Pré-condições:** operador administrador; módulo habilitado.
- **Passos:** 1. Executar uma escrita (assumir/transferir/resolver, criar número, salvar SLA, etc.). 2. Abrir `/whatsapp/auditoria`. 3. Filtrar por usuário/ação/objeto/período. 4. Paginar.
- **Resultado esperado:** registro aparece com usuário, ação, objeto, objetoId, alteração e data/hora; paginação por `pagina`/`tamanho` e envelope `items`/`totalCount`.
- **Rotas/endpoints/tabelas:** `/whatsapp/auditoria`; `GET /api/v1/whatsapp/auditoria`; `tbwhatsapp_auditoria`.
- **Status/automação:** **cenário documentado**.

