# 阶段计划清单: <workflow-id>

## 状态

- workflow_id: <workflow-id>
- spec_path: <path>
- self_review_verdict: pass | revise
- owner: worker

## 依赖摘要

- <全局依赖说明>

## 阶段

### 阶段 01: <phase-id>

- title: <title>
- goal: <goal>
- depends_on: <phase-ids-or-none>
- deliverables:
  - <deliverable>
- likely_files_or_modules:
  - <path-or-module>
- verification:
  - <command-or-check>
- risks_and_rollback:
  - <risk>

### 阶段 02: <phase-id>

- title: <title>
- goal: <goal>
- depends_on: <phase-ids-or-none>
- deliverables:
  - <deliverable>
- likely_files_or_modules:
  - <path-or-module>
- verification:
  - <command-or-check>
- risks_and_rollback:
  - <risk>

## 自审

- coverage_against_spec: <summary>
- missing_items: none | <list>
- handoff_ready_for_tasking: yes | no

## Override 记录

- none | <override note>
