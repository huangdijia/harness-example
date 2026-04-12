# 调度 Prompt: `phaseReview`

你是 `worker`。当前 job 是 `phaseReview`。

## 任务范围

- workflow_id: `{{workflow_id}}`
- job_id: `{{job_id}}`
- phase_id: `{{phase_id}}`
- 主要读取：
  - `{{phase_plan_list_path}}`
  - `{{phase_task_list_path}}`
  - `{{phase_result_path}}`
- 可选读取：
  - 只有验证结论必须依赖 repo 事实时，才读 `{{repo_root}}` 下对应文件
- 写入：
  - `{{phase_review_path}}`
  - `{{result_file_path}}`

## 核心规则

- 默认只读三个输入 artifact
- 不要读无关文件
- 如果 `{{phase_review_path}}` 已存在，直接整文件覆盖

## 必做步骤

1. 读 phase plan、phase task list、phase result
2. 对照 deliverable、verification、实际结果做 review
3. 写 `{{phase_review_path}}`
4. 写 `{{result_file_path}}`

## Phase review 必须包含

- workflow_id
- phase_id
- status: `pass` | `fail`
- reviewed_artifacts
- deliverables_present
- verification_sufficient
- drift_from_phase_goal
- findings
- next action

## 审查重点

- deliverables 是否存在
- `verification_run` 是否真的支撑功能完成
- `scope_verification` 是否真的支撑“业务改动未越出 write scope”
- summary 不能声称超过证据本身

## Scope 判定规则

- 只审业务改动路径，也就是 `changed_files_or_artifacts`
- run 目录内 transport 文件不算业务越界：
  - `prompt.md`
  - `stdout.jsonl`
  - `stderr.log`
  - `result.json`
- 如果 `scope_verification` 已经证明 `changed_files_or_artifacts` 都在允许前缀内，就可以视为 scope 证据充分

## 阻塞规则

只有这几种情况才返回 `status: blocked`：

- 任一必需输入 artifact 缺失
- 输入 artifact 内容不足以形成有效 review

能给出 `fail` 时，不要返回 `blocked`。

## 输出要求

- review 文档格式按 `docs/templates/phase-review-template.md`
- 结果 JSON 格式按 `docs/templates/worker-result-template.json`

## 结果 JSON 示例

```json
{
  "status": "done",
  "job": "phaseReview",
  "workflow_id": "{{workflow_id}}",
  "phase_id": "{{phase_id}}",
  "started_at": "2026-04-12T10:00:00Z",
  "finished_at": "2026-04-12T10:03:00Z",
  "exit_code": 0,
  "artifact_write_ok": true,
  "retryable": false,
  "verdict": "pass",
  "outputs": [
    {
      "artifact": "phaseReview",
      "path": "{{phase_review_path}}"
    }
  ],
  "next_action": "proceed_to_next_phase",
  "summary": "<one sentence>",
  "blockers": []
}
```
