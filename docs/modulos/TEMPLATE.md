---
id: mod.<nome>
titulo: "<Nome do módulo>"
tipo: modulo
area: <area>
sistema: central
tags: [<tag>, <tag>]
dono: <pessoa>
status: ativo
revisado_em: "YYYY-MM-DD"
fonte:
  - caminho/arquivo.cs
testes:
  - caminho/Teste.cs
tokens_estimados: 1200
---

# <Nome do módulo>

> Este template é o contrato das fichas de `docs/modulos/`. Copie o arquivo,
> substitua o frontmatter e cada seção. A ficha **aponta**; não duplica o
> documento profundo.
>
> 1. Toda linha de rota, endpoint, tabela, serviço e regra cita **arquivo** ou
>    **`arquivo#símbolo`** (nunca número de linha).
> 2. Fato não confirmado vai para a seção 14 (Pendências), não para o corpo.
> 3. Não copie texto longo de `docs/referencia/`, `docs/arquitetura/` ou
>    `docs/operacao/`: resuma e aponte.
> 4. Sem hash, contagem, data ou valor que não venha de fonte lida.
> 5. Em divergência entre documentos, prevalece o temático; registre na seção 14.
> 6. A ficha liga o glossário único (`docs/glossario.md`) e não cria termos.

## 1. Quando usar este card

<!-- Gatilhos e palavras-chave que devem fazer o agente abrir esta ficha. -->

## 2. Propósito e escopo

<!-- 3–6 linhas: o que o módulo resolve e o que NÃO cobre. Fonte: docs/referencia/regras-negocio.md. -->

## 3. Rotas do frontend

| URL | Componente | Observação | Fonte |
|---|---|---|---|
| | | | |

## 4. Endpoints do backend

| Método | Rota | Autorização | Observação | Fonte |
|---|---|---|---|---|
| | | | | |

## 5. Tabelas e entidades

| Classe | Tabela | Papel | Fonte |
|---|---|---|---|
| | | | |

## 6. Serviços e lógica

| Camada | Serviço/peça | Responsabilidade | Fonte |
|---|---|---|---|
| backend | | | |
| frontend | | | |

## 7. Regras de negócio

<!-- Bullets curtos, cada um com fonte. Regras que um dev quebra sem perceber entram primeiro. -->

## 8. Autorização

<!-- Policies, claims, gates, roles e o espelho no frontend. Diga quem é a fonte da verdade. -->

## 9. Onde alterar

| Intenção | Backend (arquivo#símbolo) | Frontend (arquivo#símbolo) | Teste |
|---|---|---|---|
| | | | |

## 10. Contratos que não podem quebrar

<!-- Invariantes, efeitos colaterais, autenticação, auditoria, formatos. -->

## 11. Armadilhas conhecidas

<!-- Bugs recorrentes, compatibilidade 100, OPENJSON, claims, soft delete, cache, etc. -->

## 12. Testes

<!-- Automatizados (backend/frontend) e cenários QA. Diferencie "validado" de "documentado". -->

## 13. Orçamento de leitura

- Abra primeiro: esta ficha.
- Abra depois: somente os arquivos da seção 9 e o `INDEX.md` profundo citado.
- Não abra: os documentos profundos inteiros; use a seção indicada.
- Limite: **1 ficha**, **4 trechos de código** e **≤ 12 tool calls** por tarefa.

## 14. Pendências

<!-- Fatos por confirmar, divergências e lacunas. Vazio = (nenhuma). -->
