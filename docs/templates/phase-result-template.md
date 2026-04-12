# 阶段结果: <phase-id>

## 状态

- workflow_id: <workflow-id>
- phase_id: <phase-id>
- phase_task_list_path: <path>
- execution_status: ready | in_progress | blocked | complete
- owner: worker

## 任务执行日志

### 任务 01: <task-id>

- status: pending | in_progress | complete | blocked
- summary: <发生了什么>
- changed_files_or_artifacts:
  - <path>
- verification_run:
  - <command-or-check>
- scope_verification:
  - none | <boundary proof>
- follow_up_notes: none | <notes>

### 任务 02: <task-id>

- status: pending | in_progress | complete | blocked
- summary: <发生了什么>
- changed_files_or_artifacts:
  - <path>
- verification_run:
  - <command-or-check>
- scope_verification:
  - none | <boundary proof>
- follow_up_notes: none | <notes>

## 阶段摘要

- blockers: none | <list>
- next_action: continue | wait_for_cc | ready_for_review
- verification_summary: <summary>
