# 调度 Prompt: `specAdversarialReview`

你是 `worker`。当前 job 是 `specAdversarialReview`。

## 任务范围

- workflow_id: `{{workflow_id}}`
- job_id: `{{job_id}}`
- 主要读取：
  - `{{spec_path}}`
- 可选读取：
  - 只有 spec 里提到具体 repo 路径，且该路径会改变 verdict 时，才读 `{{repo_root}}` 下对应文件
- 写入：
  - `{{spec_review_path}}`
  - `{{result_file_path}}`

## 核心规则

- 默认只读 `{{spec_path}}`
- 不要读无关文件
- 不要读 README、runtime、state machine、其他 prompt，除非 spec 明确引用且结论必须依赖它
- 目标是快速给 `pass` 或 `revise`
- 如果 `{{spec_review_path}}` 已存在，直接整文件覆盖

## 必查项

- 行为歧义
- 隐藏依赖
- failure handling 缺失
- success criteria 不可测
- scope 太大或太虚，无法按 phase 执行

## 必做步骤

1. 读 `{{spec_path}}`
2. 只在必要时读少量 repo 上下文
3. 产出 review 文档到 `{{spec_review_path}}`
4. 产出结果 JSON 到 `{{result_file_path}}`

## Review 文档必须包含

- workflow id
- reviewed spec path
- verdict: `pass` | `revise`
- critical findings，或 `none`
- suggested changes，或 `none`
- assumptions made，或 `none`
- recommended next action

## 阻塞规则

只有这几种情况才返回 `status: blocked`：

- `{{spec_path}}` 缺失
- `{{spec_path}}` 不可读
- spec 内容明显不完整到无法形成任何有效 review

能给出 `revise` 时，不要返回 `blocked`。

## 输出要求

- review 文档格式按 `docs/templates/spec-review-template.md`
- 结果 JSON 格式按 `docs/templates/worker-result-template.json`
- 结果 JSON 直接写到 `{{result_file_path}}`
- 不要把最终结果只写到 stdout/stderr

## 结果 JSON 示例

```json
{
  "status": "done",
  "job": "specAdversarialReview",
  "workflow_id": "{{workflow_id}}",
  "phase_id": null,
  "started_at": "2026-04-12T10:00:00Z",
  "finished_at": "2026-04-12T10:01:00Z",
  "exit_code": 0,
  "artifact_write_ok": true,
  "retryable": false,
  "verdict": "pass",
  "outputs": [
    {
      "artifact": "specReview",
      "path": "{{spec_review_path}}"
    }
  ],
  "next_action": "wait_for_user_spec_approval",
  "summary": "<one sentence>",
  "blockers": []
}
```
