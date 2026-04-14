# CC Leader Worker Prompt 合同

这个文档定义 `cc-leader` 里所有 worker job 的共享 prompt 合同。

## 目标

让每个 worker job 都：

- 能从 manifest 发现
- 能被 harness 无猜测渲染
- 能被 `cc` 稳定解析结果

当前 worker runtime 只支持 `codex-cli`。

## 发现方式

对每个 job，读：

- `workerJobs.<job>.dispatch.promptContractPath`
- `workerJobs.<job>.dispatch.resultFileVariable`
- `workerJobs.<job>.dispatch.requiredVariables`
- `workerJobs.<job>.timeoutSeconds`
- `workerJobs.<job>.writesCode`

prompt 模板文件都在 `docs/prompts/`。

## 渲染规则

dispatch 前：

1. 从 `promptContractPath` 读取模板
2. 替换 `requiredVariables` 里全部 `{{variable}}`
3. 所有文件路径变量都传绝对路径
4. 只要还有未替换占位符，就不要 dispatch

可选变量可以加，但必需变量必须全有。

多行变量，比如 `write_scope`、`phase_review_paths`，要渲染成 markdown 预格式化行，不要塞 JSON。prompt 文件就是这样预期的。

`result_file_path` 也必须由 `cc` 预先生成，并传绝对路径。

## Worker 规则

每个 worker prompt 都遵守：

- 不和最终用户对话
- 只在 job scope 内行动
- 只读声明过的输入 artifact 和必要 repo 上下文
- 如果输入足够且未遇到真实 blocker，要一次完成整个 job，不要把正常中间进度当作提前结束点，也不要询问用户是否继续
- 结束前必须写声明过的输出 artifact
- 结束前必须写 `result_file_path`
- 阻塞时不要编造输入
- 必须遵守 `docs/artifact-rules.md` 和 `docs/verification-contract.md`
- 如果输出 artifact 或 `result_file_path` 已存在，直接覆盖为当前 job 的新结果，不要保留旧内容

## 结果文件合同

worker 写完 artifact 后，必须把单个 JSON 对象写到 `result_file_path`，格式：

```json
{
  "status": "done | blocked",
  "job": "worker job name",
  "workflow_id": "workflow id",
  "phase_id": "phase id or null",
  "started_at": "ISO-8601",
  "finished_at": "ISO-8601",
  "exit_code": 0,
  "artifact_write_ok": true,
  "recovered_from_artifact": false,
  "retryable": false,
  "verdict": "job-specific verdict or null",
  "outputs": [
    {
      "artifact": "artifact type",
      "path": "/absolute/path"
    }
  ],
  "next_action": "short coordinator action",
  "summary": "one-sentence summary",
  "blockers": []
}
```

## 字段语义

- `status`
  - `done`: job 已完成，且声明产物已写出
  - `blocked`: job 无法安全完成
- `artifact_write_ok`
  - 只有当输出 artifact 真的写出时才是 `true`
- `recovered_from_artifact`
  - `true` 表示 worker 没写结果文件，结果文件由 harness 根据 artifact 补写
- `retryable`
  - `true` 表示同类 job 在输入变化后可再试
- `verdict`
  - job 语义结果，比如 `pass`、`revise`、`fail`、`complete`
- `outputs`
  - 只列真实写出的文件
- `blockers`
  - `done` 时应为空数组
  - `blocked` 时应有一个或多个短 blocker 字符串

## CC 解析规则

worker 进程退出后：

1. 读取 `result_file_path`
2. 解析结果 JSON
3. 校验声明过的输出文件都存在
4. 从磁盘读取产物
5. 下一步跳转同时看：
   - 结果文件里的 transport 状态
   - artifact 正文里的语义 verdict

stdout/stderr 不作为状态真源。

## 参考

- transport 合同：`docs/transport-contract.md`
- runtime 合同：`docs/runtime-contract.md`
- state machine：`docs/state-machine.md`
- 产物规则：`docs/artifact-rules.md`
- verification 合同：`docs/verification-contract.md`
- 结果文件模板：`docs/templates/worker-result-template.json`
- 当前 worker CLI：`codex exec`

## Prompt 清单

- `specAdversarialReview`: `docs/prompts/spec-adversarial-review.md`
- `phasePlanSynthesis`: `docs/prompts/phase-plan-synthesis.md`
- `phaseTaskSynthesis`: `docs/prompts/phase-task-synthesis.md`
- `phaseExecution`: `docs/prompts/phase-execution.md`
- `phaseReview`: `docs/prompts/phase-review.md`
- `finalSpecReview`: `docs/prompts/final-spec-review.md`
