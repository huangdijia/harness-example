# Plan: CC Leader Harness 加固

## 目标

根据评估报告修复 5 个已知缺口, 提升 cc-leader 到可用于有人值守生产场景的水平。

## 变更范围

只改 `scripts/cc-leader-harness.mjs`。不改 skill、合同、prompt、模板、样例。

---

## Task 1: 加 rollback anchor

**文件**: `scripts/cc-leader-harness.mjs`
**位置**: `dispatch` 命令, codex 调用前
**改动**:
- 在 `runCodex` 调用前, 检查 `manifest.workerJobs[jobName].writesCode`
- 如果 `true`, 执行 `git tag pre-<job-id>` 作为 rollback 锚点
- 用 `spawnSync("git", ["tag", "pre-" + jobContext.jobId])` 实现
- tag 创建失败 (例如 dirty worktree) 只 warn, 不 block dispatch

**验证**: `git tag -l 'pre-*'` 能看到 tag

## Task 2: 加 state schema 校验

**文件**: `scripts/cc-leader-harness.mjs`
**位置**: `readState` 函数之后, 新增 `validateState` 函数
**改动**:
- 新增 `validateState(state)` 函数
- 检查关键字段类型:
  - `workflow_id`: string | null
  - `current_phase`: string
  - `spec_approved`, `spec_review_passed`, `phase_plan_review_passed`: boolean
  - `ready_phase_ids`, `completed_phase_ids`, `reviewed_phase_ids`: array
  - `job_attempts`: object
- 类型不匹配时 `fail()` 并报错字段名和实际类型
- 在 `readState` 返回前调用, 所有命令自动过校验

**验证**: `state:set --set spec_approved=1` 后再读 state 应该报错 (1 不是 boolean)

## Task 3: 丰富 report 命令

**文件**: `scripts/cc-leader-harness.mjs`
**位置**: `report` 命令的 final report 生成部分
**改动**:
- 读 spec 文件, 提取 `## 目标` 段落正文 (不只是 "见 spec_path")
- 读每个 phase result, 提取 `## 阶段摘要` 段落
- 读 final spec review, 提取 `## 结论` 段落
- 把提取内容填入 final report 对应位置
- 新增 `extractSection(filePath, heading)` 辅助函数: 从 markdown 提取 `## heading` 到下一个 `##` 之间的内容

**验证**: 生成的 final report 里应包含 spec 目标原文, 而非 "见 xxx"

## Task 4: 加 stale lock 清理

**文件**: `scripts/cc-leader-harness.mjs`
**位置**: `withStateLock` 函数
**改动**:
- 在 `mkdir` 失败 (EEXIST) 时, 检查 lock 目录的 mtime
- 如果 lock 目录存在超过 5 分钟, 输出 warn "State lock 已超 5 分钟, 疑似残留, 自动清理"
- 删除 stale lock 目录, 继续尝试获取
- 只在首次检测到 stale 时清理一次, 避免竞态

**验证**: 手动创建 lock 目录, 修改 mtime 到 6 分钟前, 跑 `state:get` 应该 warn 并自动恢复

## Task 5: smoke dry-run 模式

**文件**: `scripts/smoke-run.mjs`
**位置**: 新增 `--dry-run` 参数支持
**改动**:
- 解析 `process.argv` 检查 `--dry-run`
- dry-run 模式下, 不调 `codex exec`, 改为从 `docs/examples/minimal-todo/` 复制对应 artifact 作为 worker 输出
- 具体: 每个 `runNode(dispatchArgs(...))` 改为 `runNode(dispatchArgs(..., ["--dry-run"]))` 或直接跳过 dispatch, 用 harness 的 `prepare-job` 生成 prompt 后, 手动写固定 result.json + 复制 example artifact
- smoke 结束后仍检查完整 state 流转和 artifact 存在性
- 输出末尾标注 `(dry-run: 未调用 codex, 使用固定 artifact)`

**验证**: `npm run smoke -- --dry-run` 应在无 codex 环境下通过

---

## 执行方式

按评估报告优先级, Task 1-4 改 harness, Task 5 改 smoke。全部在 `scripts/` 内完成, 通过 codex-cli dispatch。

每个 task 完成后跑 `node --check scripts/cc-leader-harness.mjs` 和 `npm run validate` 确保不破坏现有功能。

## 最终验证

- `npm run validate` 通过
- `npm run smoke -- --dry-run` 通过
- `node --check scripts/cc-leader-harness.mjs` 通过
- `node --check scripts/smoke-run.mjs` 通过
