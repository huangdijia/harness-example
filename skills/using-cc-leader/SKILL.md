---
name: using-cc-leader
description: 每个会话或 turn 开头都用。识别当前 cc workflow phase，并路由到正确 skill。
---

# 使用 CC Leader

这是 `cc-leader` 的 bootstrap skill。

职责：

1. 看用户请求、session state、artifact
2. 判断当前 phase
3. 在做实质动作前，路由到正确 phase skill

## 指令优先级

顺序：

1. 用户直接指令
2. 已批准 workflow artifact
3. `cc-leader` phase skill
4. 默认助手行为

用户明确 override gate，就记录并继续。

## 核心规则

在没判断 phase 前，不要写代码、计划、报告。

按状态路由：

- 没有已批准 spec -> `drafting-specs`
- spec 已批准，但没有 phase plan list -> `writing-phase-plans`
- phase plan list 已有，但缺 phase task list -> `writing-phase-tasks`
- 已有 ready 的 phase task list，且还有 phase 没执行完 -> `executing-phase-tasks`
- 某个已完成 phase 还没 review，或 final spec review 还没做 -> `reviewing-phase-results`
- 所有 phase 都 review 完，且 final spec review 通过 -> `reporting-results`

## Gate 规则

默认 gate：

- spec 批准前，不写 phase plan
- phase review 过前，不开下一个依赖 phase 执行
- final spec review 过前，不出最终报告

除非用户明确要求，否则不要跳 gate。

## 会话合同

追踪或推断这些字段：

- `workflow_id`
- `current_phase`
- `spec_path`
- `spec_review_path`
- `spec_review_passed`
- `spec_approved`
- `phase_plan_list_path`
- `phase_plan_review_passed`
- `phase_task_dir`
- `phase_result_dir`
- `phase_review_dir`
- `ready_phase_ids`
- `active_phase_id`
- `completed_phase_ids`
- `reviewed_phase_ids`
- `final_spec_review_path`
- `final_report_path`
- `active_job_id`
- `last_result_file_path`
- `override_log_path`
- `job_attempts`
- `current_execution_root`

如果没有结构化 state，就根据明确 approval、artifact 是否存在、最近 review 结果来推断。

## 响应模式

开工时：

1. 说当前用哪个 `cc-leader` skill
2. 说为什么
3. 按该 skill 执行

示例：

- "Using `drafting-specs`，因为现在还没有已批准 spec。"
- "Using `writing-phase-tasks`，因为已有 phase plan list，现在要按 phase 产 task list。"
- "Using `reviewing-phase-results`，因为某个 phase 刚执行完，必须对照 plan 审。"

## 反模式

- 粗糙请求直接进 execution
- 没书面 spec 就 planning
- 上个依赖 phase 还没 review 就跑后续 phase
- final spec review 没过就宣称完成
