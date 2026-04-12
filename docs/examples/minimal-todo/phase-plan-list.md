# 阶段计划清单: wf-minimal-todo-20260412-ab12

## 状态

- workflow_id: wf-minimal-todo-20260412-ab12
- spec_path: docs/examples/minimal-todo/spec.md
- self_review_verdict: pass
- owner: worker

## 依赖摘要

- 只有一个 phase，无跨 phase 依赖

## 阶段

### 阶段 01: phase-01-core-todo

- title: core todo
- goal: 实现 add/list 最小闭环
- depends_on: none
- deliverables:
  - CLI 可新增 todo
  - CLI 可列出 todo
- likely_files_or_modules:
  - src/cli.ts
  - src/store.ts
- verification:
  - `todo add "milk" && todo list`
- risks_and_rollback:
  - JSON 存储格式变更要可回滚

## 自审

- coverage_against_spec: spec 要求已覆盖
- missing_items: none
- handoff_ready_for_tasking: yes

## Override 记录

- none
