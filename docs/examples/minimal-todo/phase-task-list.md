# 阶段任务清单: phase-01-core-todo

## 状态

- workflow_id: wf-minimal-todo-20260412-ab12
- phase_id: phase-01-core-todo
- phase_plan_path: docs/examples/minimal-todo/phase-plan-list.md
- self_review_verdict: pass
- owner: worker

## 阶段目标

实现 add/list 最小闭环。

## 任务

### 任务 01: task-01-add-command

- objective: 实现 add 命令和存储写入
- files_or_artifacts:
  - src/cli.ts
  - src/store.ts
- ordered_steps:
  - 增加 add 命令分支
  - 增加写入 store 方法
- verification:
  - `todo add "milk"`
- done_definition: add 后存储文件包含新 todo
- rollback_or_retry_note: 失败时回退命令分支和写入逻辑

### 任务 02: task-02-list-command

- objective: 实现 list 命令和存储读取
- files_or_artifacts:
  - src/cli.ts
  - src/store.ts
- ordered_steps:
  - 增加 list 命令分支
  - 增加读取 store 方法
- verification:
  - `todo list`
- done_definition: list 输出包含刚添加的 todo
- rollback_or_retry_note: 失败时回退读取和输出逻辑

## 自审

- deliverables_covered: yes
- dependency_order_valid: yes
- blockers_or_assumptions: none
- ready_for_execution: yes

## Override 记录

- none
