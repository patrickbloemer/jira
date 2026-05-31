# Doc canônico — Módulo Jira

> **Fonte de verdade** do módulo Jira (backlog interno). Define modelo de dados,
> máquina de estados, API e regras de negócio de forma **agnóstica de estética** —
> cada sistema de destino usa o próprio design system.
>
> Esta prosa acompanha o `registry.json` (regras versionadas, máquina-legível).
> **Camada 1** = declarativa (aqui). **Camada 2** = referência de implementação
> (paths + SQL do Sistema A: Nexus), dentro de cada regra no `registry.json`.
>
> **ESCOPO:** o doc descreve **só features do módulo Jira** (comportamento, UI, máquina
> de estados, entidades). **Infra de host** — auth, multi-tenant/`org_id`, RLS, papéis
> de acesso — **não entra como regra**; é decisão de cada sistema. O `org_id`/RLS que
> aparece na **camada 2** é só referência da implementação do Nexus.

- **Versão:** 2.1.0
- **Sistema A (referência):** Nexus — `github.com/patrickbloemer/nexus`
- **Prefixo DB:** `backlog_` · **Label UI:** "Jira"

---

## Como ler / versionar

- Cada regra de comportamento vive no `registry.json` como `RULE-NNN`, com `since`
  (versão em que entrou).
- Mudou um comportamento? Bump da `version`, nova/atualizada RULE com `since` novo.
- A prosa abaixo é o índice navegável das regras; o detalhe normativo está no JSON.

---

## 1. Conceitos centrais

Task · Subtask (1 nível) · Sprint · Feedback de teste (auditoria imutável).
_(detalhar conforme evoluir)_

## 2. Modelo de dados

Soft delete (`deleted_at`) em tudo.
Tabelas: `backlog_tasks`, `backlog_subtasks`, `backlog_sprints`,
`backlog_sprint_tasks`, `backlog_task_feedback`, `backlog_task_attachments`,
`backlog_task_counters`.
→ RULE-001, RULE-006, RULE-008, RULE-010, RULE-011

## 3. Máquina de estados

`maturing → ready → testing → delivered → finalizado` (task);
`open → in_progress → testing → delivered` (sprint, derivado).
→ RULE-002, RULE-003, RULE-004, RULE-005

## 4. API REST

Rotas sob `/api/backlog`. (Auth/escopo de acesso é decisão de cada sistema, fora do
escopo do doc — ver nota de ESCOPO no topo.)
→ RULE-009, RULE-011

## 5. Tela lista/board + drawers

Busca tudo + filtra no client; abas com contagem; filtro de sprint lista só sprints
vivas; toggle "agrupar por módulo" (só na lista) com cabeçalho por seção; drawer de
task com dirty-guard; drawer de sprints master/detail.
→ RULE-013

## 6. Regras invioláveis de agente

Agente nunca move pra `delivered`; `finalizado` só pela sessão Finalizar, não
selecionável como destino na UI (pills/mover-para/massa), rejeitado pela API —
mas visível em leitura como aba/coluna (ver ≠ mover).
→ RULE-012

## 7. Superfícies de UI (drawer vs modal)

Criar/editar entidade (task, sprint) acontece em **drawer** (painel lateral, dirty-guard);
confirmação destrutiva e escolha rápida (criar sprint, adicionar à sprint, descartar
mudanças) acontece em **modal** (centrado, foco preso, ESC/backdrop fecham). Descrição
agnóstica de DS: "isto é um drawer e se comporta assim" / "isto é um modal e se comporta
assim" — cada sistema mapeia pro próprio componente.
→ RULE-014, RULE-015

## 8. Anatomia da task

Catálogo normativo dos campos da task — quais existem, em que ordem aparecem, quais são
obrigatórios, quais são legados (não exibir). Mesma anatomia em A/B/C. As pílulas de
**Status** mudam por contexto (2.1.0): na **criação** o conjunto é só `maturing/ready/
testing` (delivered e finalizado ausentes — não se cria task já-entregue); na **edição**
entra `delivered` (selecionável só pelo usuário, RULE-012); `finalizado` nunca é pílula.
→ RULE-016

## 9. Tela principal (abas / filtros / tabela / board)

Abas no topo por status (labels canônicas, inclui `finalizado` só-leitura ao final);
filtros que abrem submenu com opções; tabela com colunas e dados definidos; board por
coluna de status; barra de ação em massa flutuante; menu de contexto (botão direito).
Status é dirigido **só** pelas abas — sem dropdown de status no filtro.
→ RULE-017, RULE-019

## 10. Sprints e feedback (UI)

Drawer de sprints master→detalhe + modais de criar/adicionar; seletores em pílula
(Tipo/Status/Prioridade) como toggle de seleção única; subtask de 1 nível e ciclo de
feedback de teste imutável renderizado na UI (com evidência em lightbox).
→ RULE-018, RULE-019, RULE-020

## 11. Localização (path do sistema)

Toda tela exibe, discretamente sob o título, o **path/rota atual** do sistema
(ex: `/app/jira`) — indicador de localização só-leitura, dinâmico (vem do roteador),
nunca texto fixo. Não é breadcrumb clicável.
→ RULE-021

## 12. Motion (obrigatório)

Nada é seco: toda transição (rota, drawer, modal, toast, popover, sidebar mobile,
listas/estados) é animada **via o DS**, sempre respeitando `prefers-reduced-motion`.
Durações/curvas/distâncias são do DS — B/C usam o equivalente próprio.
→ RULE-022

## 13. Aderência ao Design System (transversal)

Componentes, padrões, comportamentos, interações, animações, espaçamentos, tipografias,
cores e estados seguem **rigorosamente o DS oficial** de cada projeto. Proibido criar
variações quando já existe padrão no DS; em conflito, prevalece o DS. Consistência
visual e comportamental é requisito obrigatório, não recomendação. Governa **como**
todas as outras regras de UI deste doc são implementadas.
→ RULE-023

## 14. Atalhos globais + deep-link

O Jira é alcançável de qualquer rota: tecla `n` abre o drawer de criação de task
sobreposto; tecla `j` abre uma lista read-only "Prontas no Jira" (status `ready`);
`Cmd/Ctrl+K` abre o command-palette que indexa tasks/sprints e faz deep-link
(`?task=<id>` abre a task, `?sprints=1` abre o drawer de sprints — a tela consome e
limpa o param). Atalhos ignoram campos editáveis (exceto o palette). B/C mapeiam as
teclas/superfícies pro próprio padrão; o conteúdo normativo é criar/consultar/abrir o
Jira de qualquer lugar.
→ RULE-024

## 15. Histórico de auditoria + desfazer

Toda mudança em task/subtask/sprint vira uma linha imutável num log de auditoria
(gatilho no banco, com snapshot antes/depois). Um drawer "Histórico" mostra as entradas
em ordem reversa com o diff `de → para` dos campos relevantes; updates de task têm botão
**Restaurar** que reaplica o snapshot anterior (inclusive des-deletar). Guardas
invioláveis: nunca restaura pro estado `finalizado` (RULE-012); pós-restauração
ressincroniza o status da sprint (RULE-005). Existe pra tornar reversível um bulk
acidental.
→ RULE-025

## 16. Mapa de posição (layout normativo) — arranjo ≠ estética

A partir de 2.0.0 o doc fixa **onde** os componentes aparecem e **como** se arranjam
entre si — separado da **estética** (aparência), que segue o DS de cada sistema. Regra
de ouro: _"o componente você serve do jeito do seu DS — 100% sua estética —, mas o
posicionamento/arranjo é este."_ Medidas em px na camada 2 são referência do Nexus, não
obrigatórias; o normativo é a relação (em linha / empilhado / ancorado / compacto).

**Tela principal** — pilha vertical: cabeçalho (título+path à esquerda, ações
[Histórico][Sprints][Nova task] à direita) → faixa de abas (contêiner único, segmentos
de largura igual) → **barra de filtros = UMA faixa horizontal compacta, controles em
linha auto-width** (busca·Tipo·Módulo·Subtasks·Sprint·Ordenar, e Lista/Board·Agrupar·
Limpar·contador ancorados à direita); é uma toolbar compacta, não uma coluna de campos
full-width → conteúdo (tabela de largura total / board em N colunas) → barra de ação em
massa flutuante, centrada na área de conteúdo.

**Drawer** — colado numa borda (direita em A), full-height, três regiões empilhadas:
**header concentrado** (cluster esquerdo = identidade [código + badges Tipo/Módulo/
Prioridade]; cluster direito = **kebab ⋯ + X**, com Duplicar/Sprint/**Excluir dentro do
kebab** — destrutivo nunca solto) → corpo em coluna única, campos empilhados na ordem
fixa de RULE-016 → rodapé (metadados à esquerda, [Cancelar][Salvar] à direita).
**Modais** são centrados (não deslizam de borda).
→ RULE-026, RULE-027

---

## Índice de regras (registry.json)

| ID       | since | Título                                                   |
| -------- | ----- | -------------------------------------------------------- |
| RULE-001 | 1.0.0 | Código curto sequencial por org (TSK-NNN)                |
| RULE-002 | 1.0.0 | Máquina de estados de status da task (+ labels canônicas) |
| RULE-003 | 1.0.0 | Plano de teste obrigatório ao mover pra testing          |
| RULE-004 | 1.0.0 | Ciclo de feedback de teste (imutável)                    |
| RULE-005 | 1.0.0 | Status da sprint é derivado das tasks                    |
| RULE-006 | 1.0.0 | Soft delete em tudo                                      |
| RULE-008 | 1.0.0 | Prioridade simples (baixa/media/alta) + priority_rank    |
| RULE-009 | 1.0.0 | Sprint: code SPR-NNN, name=code, criação transacional    |
| RULE-010 | 1.0.0 | Subtasks de 1 nível                                      |
| RULE-011 | 1.0.0 | Anexos: paste de imagem → upload (nunca base64 inline)   |
| RULE-012 | 1.1.0 | Regras invioláveis de agente (delivered / finalizado)    |
| RULE-013 | 1.1.0 | Lista: busca tudo + filtra no client + filtros persist.  |
| RULE-014 | 1.2.0 | Superfícies de UI: o que abre em DRAWER vs MODAL         |
| RULE-015 | 1.2.0 | Confirmação destrutiva + guarda de descarte (dirty-guard)|
| RULE-016 | 1.2.0 | Anatomia da task: campos, ordem e obrigatoriedade        |
| RULE-017 | 1.2.0 | Tela principal: abas/filtros/tabela/board/massa/contexto |
| RULE-018 | 1.2.0 | Drawer de Sprints (master→detalhe) e modais de sprint    |
| RULE-019 | 1.2.0 | Seletores em pílula (Tipo/Status/Prioridade)             |
| RULE-020 | 1.2.0 | Subtask e ciclo de feedback de teste, na UI              |
| RULE-021 | 1.3.0 | Path do sistema discreto sob o título da tela            |
| RULE-022 | 1.3.0 | Motion obrigatório em toda transição (via DS)            |
| RULE-023 | 1.3.0 | Aderência ao Design System (regra obrigatória)           |
| RULE-024 | 1.4.0 | Atalhos globais do Jira + deep-link (palette / n / j)    |
| RULE-025 | 1.4.0 | Histórico de auditoria + desfazer (restaurar estado)     |
| RULE-026 | 2.0.0 | Mapa de posição: princípio + tela principal              |
| RULE-027 | 2.0.0 | Mapa de posição: anatomia do drawer + modais             |
