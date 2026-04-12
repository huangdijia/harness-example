# 调度 Prompt: `phasePlanSynthesis`

你是 `worker`。当前 job 是 `phasePlanSynthesis`。

## 任务范围

- workflow_id: `{{workflow_id}}`
- job_id: `{{job_id}}`
- 主要读取：
  - `{{spec_path}}`
- 可选读取：
  - 只有 spec 提到具体 repo 路径，且会影响 phase 边界时，才读 `{{repo_root}}` 下对应文件
- 写入：
  - `{{phase_plan_list_path}}`
  - `{{result_file_path}}`

## 核心规则

- 默认只读 `{{spec_path}}`
- 不要读无关文件
- 目标是快速产出可执行 phase plan list
- 如果 `{{phase_plan_list_path}}` 已存在，直接整文件覆盖

## 必做步骤

1. 读 `{{spec_path}}`
2. 拆出最小且有意义的 phase 顺序
3. 明确每个 phase 的 deliverable、verification、depends_on
4. 写 `{{phase_plan_list_path}}`
5. 做 self review
6. 写 `{{result_file_path}}`

## Phase plan 必须包含

- 稳定 phase_id — 格式**必须**是 `phase-<NN>-<slug>`
  - `<NN>`: 两位数字序号，从 01 开始（01, 02, 03 …）
  - `<slug>`: 纯小写字母+数字，用 `-` 分隔，不能以 `-` 结尾
  - 正则: `^phase-[0-9]{2}-[a-z0-9]+(?:-[a-z0-9]+)*$`
  - 正确示例: `phase-01-core-todo`, `phase-02-auth-integration`
  - 错误示例: `phase-1-todo`, `phase-01-Core-Todo`, `phase-01-`
- title
- goal
- depends_on
- deliverables
- likely files 或 artifacts
- verification
- risks_and_rollback
- self_review_verdict

## Verdict

- `pass`: phase 覆盖 spec，顺序成立，可直接 tasking
- `revise`: phase 边界、覆盖、验证或依赖不够清楚

## 阻塞规则

只有这几种情况才返回 `status: blocked`：

- `{{spec_path}}` 缺失
- `{{spec_path}}` 不可读
- spec 明显自相矛盾到无法拆 phase

能给出 `revise` 时，不要返回 `blocked`。

## 输出要求

- phase plan 文档格式按 `docs/templates/phase-plan-list-template.md`
- 结果 JSON 格式按 `docs/templates/worker-result-template.json`
- 结果 JSON 直接写到 `{{result_file_path}}`

## 结果 JSON 示例

```json
{
  "status": "done",
  "job": "phasePlanSynthesis",
  "workflow_id": "{{workflow_id}}",
  "phase_id": null,
  "started_at": "2026-04-12T10:00:00Z",
  "finished_at": "2026-04-12T10:05:00Z",
  "exit_code": 0,
  "artifact_write_ok": true,
  "retryable": false,
  "verdict": "pass",
  "outputs": [
    {
      "artifact": "phasePlanList",
      "path": "{{phase_plan_list_path}}"
    }
  ],
  "next_action": "dispatch_phase_task_synthesis",
  "summary": "<one sentence>",
  "blockers": []
}
```
