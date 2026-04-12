# 阶段任务清单: <phase-id>

## 状态

- workflow_id: <workflow-id>
- phase_id: <phase-id>
- phase_plan_path: <path>
- self_review_verdict: pass | revise
- owner: worker

## 阶段目标

<从 phase plan 重述目标>

## 任务

### 任务 01: <task-id>

- objective: <objective>
- files_or_artifacts:
  - <path>
- ordered_steps:
  - <step>
- verification:
  - <command-or-check>
- done_definition: <done condition>
- rollback_or_retry_note: <note-or-none>

### 任务 02: <task-id>

- objective: <objective>
- files_or_artifacts:
  - <path>
- ordered_steps:
  - <step>
- verification:
  - <command-or-check>
- done_definition: <done condition>
- rollback_or_retry_note: <note-or-none>

## 自审

- deliverables_covered: yes | no
- dependency_order_valid: yes | no
- blockers_or_assumptions: none | <list>
- ready_for_execution: yes | no

## Override 记录

- none | <override note>
