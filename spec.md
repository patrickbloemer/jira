# Doc canônico — Módulo Jira

> **Fonte de verdade** do módulo Jira (backlog interno). Define modelo de dados,
> máquina de estados, API e regras de negócio de forma **agnóstica de estética** —
> cada sistema de destino usa o próprio design system.
>
> Esta prosa acompanha o `registry.json` (regras versionadas, máquina-legível).
> **Camada 1** = declarativa (aqui). **Camada 2** = referência de implementação
> (paths + SQL do Sistema A: Nexus), dentro de cada regra no `registry.json`.

- **Versão:** 1.0.0
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

Multi-tenant (`org_id` em tudo), soft delete (`deleted_at`) em tudo.
Tabelas: `backlog_tasks`, `backlog_subtasks`, `backlog_sprints`,
`backlog_sprint_tasks`, `backlog_task_feedback`, `backlog_task_attachments`,
`backlog_task_counters`.
→ RULE-001, RULE-006, RULE-007, RULE-008, RULE-010, RULE-011

## 3. Máquina de estados

`maturing → ready → testing → delivered → finalizado` (task);
`open → in_progress → testing → delivered` (sprint, derivado).
→ RULE-002, RULE-003, RULE-004, RULE-005

## 4. API REST

Rotas sob `/api/backlog`, só funcionário, `org_id` do token.
→ RULE-007, RULE-009, RULE-011

## 5. Tela lista/board + drawers

Busca tudo + filtra no client; abas com contagem; drawer de task com dirty-guard;
drawer de sprints master/detail.
→ RULE-013

## 6. Regras invioláveis de agente

Agente nunca move pra `delivered`; `finalizado` só pela sessão Finalizar, não
selecionável na UI, rejeitado pela API.
→ RULE-012

---

## Índice de regras (registry.json)

| ID       | since | Título                                                   |
| -------- | ----- | -------------------------------------------------------- |
| RULE-001 | 1.0.0 | Código curto sequencial por org (TSK-NNN)                |
| RULE-002 | 1.0.0 | Máquina de estados de status da task                     |
| RULE-003 | 1.0.0 | Plano de teste obrigatório ao mover pra testing          |
| RULE-004 | 1.0.0 | Ciclo de feedback de teste (imutável)                    |
| RULE-005 | 1.0.0 | Status da sprint é derivado das tasks                    |
| RULE-006 | 1.0.0 | Soft delete em tudo                                      |
| RULE-007 | 1.0.0 | Multi-tenant + acesso só de funcionário                  |
| RULE-008 | 1.0.0 | Prioridade simples (baixa/media/alta) + priority_rank    |
| RULE-009 | 1.0.0 | Sprint: code SPR-NNN, name=code, criação transacional    |
| RULE-010 | 1.0.0 | Subtasks de 1 nível                                      |
| RULE-011 | 1.0.0 | Anexos: paste de imagem → upload (nunca base64 inline)   |
| RULE-012 | 1.0.0 | Regras invioláveis de agente (delivered / finalizado)    |
| RULE-013 | 1.0.0 | Lista: busca tudo + filtra no client + filtros persist.  |
