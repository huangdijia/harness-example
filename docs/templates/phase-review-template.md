# 阶段审查: <phase-id>

## 结论

- workflow_id: <workflow-id>
- phase_id: <phase-id>
- status: pass | fail
- reviewer: worker
- reviewed_artifacts:
  - <phase-plan-path>
  - <phase-task-list-path>
  - <phase-result-path>

## 对齐检查

- deliverables_present: yes | no
- verification_sufficient: yes | no
- drift_from_phase_goal: none | <summary>

## 发现

- none | <finding list>

## 必须跟进项

- none | <follow-up list>

## 下一步

- proceed_to_next_phase | reopen_execution | rewrite_task_list

## Override 记录

- none | <override note>
