# 调度 Prompt: `finalSpecReview`

你是 `worker`。当前 job 是 `finalSpecReview`，属于 `cc-leader` workflow。

## 任务范围

- workflow_id: `{{workflow_id}}`
- job_id: `{{job_id}}`
- repo_root: `{{repo_root}}`
- 读取：
  - `{{spec_path}}`
  - `{{phase_plan_list_path}}`
  - 这里列出的 phase review artifact：
    {{phase_review_paths}}
  - 只有在验证最终结论时，才读 `{{repo_root}}` 下 repo 上下文
- 写入：
  - `{{final_spec_review_path}}`
  - `{{result_file_path}}`

## 目标

站在原始已批准 spec 的视角，审整个交付结果是否真的满足目标。

## 核心规则

- 默认只读 spec、phase plan list、全部 phase review，只有验证最终结论时才读少量 repo 上下文
- 如果 `{{final_spec_review_path}}` 已存在，直接整文件覆盖
- 如果输入足够且未遇到真实 blocker，就一次完成整个 job；不要在中途停下来询问用户是否继续

## 必做步骤

1. 读已批准 spec。
2. 读 phase plan list。
3. 读 `{{phase_review_paths}}` 里列出的全部 phase review。
4. 判断：这些已完成且已通过 review 的 phase，是否满足原始 spec。
5. 用 `docs/templates/final-spec-review-template.md` 结构，把 final review 写到 `{{final_spec_review_path}}`。
6. 如有必要，先 `mkdir -p "$(dirname "{{final_spec_review_path}}")"`。
7. 把 final review verdict 设为：
   - `pass`
   - `fail`
8. 按 `docs/transport-contract.md` 和 `docs/templates/worker-result-template.json`，把结果 JSON 写到 `{{result_file_path}}`。

## 必查焦点

- requirement coverage
- unresolved gap
- evidence sufficiency
- remaining risk
- 如果不满足 spec，最小该重开哪个 phase

## 阻塞规则

如果 spec、phase plan list、或必需 phase review artifact 缺失，就返回 `status: blocked`。

## 结果文件

退出前，把单个 JSON 对象写到 `{{result_file_path}}`。

done 示例：

```json
{
  "status": "done",
  "job": "finalSpecReview",
  "workflow_id": "{{workflow_id}}",
  "phase_id": null,
  "started_at": "2026-04-12T10:00:00Z",
  "finished_at": "2026-04-12T10:02:00Z",
  "exit_code": 0,
  "artifact_write_ok": true,
  "retryable": false,
  "verdict": "pass",
  "outputs": [
    {
      "artifact": "finalSpecReview",
      "path": "{{final_spec_review_path}}"
    }
  ],
  "next_action": "proceed_to_final_report",
  "summary": "<one sentence>",
  "blockers": []
}
```
