---
name: cc-leader-drive
description: "Use when the user wants a detached Codex worker to execute an arbitrary user-specified task and keep going automatically when Codex merely asks whether to continue."
---

# 驱动 Detached Codex

这是独立于 workflow 的 drive 入口。用户明确说“开一个 detached codex 去做这个任务”“持续驱动 codex 干活”“别让它每做一点就回来问我是否继续”，或直接说 `/cc-leader-drive` 时，用这个 skill。

## 语义

- `drive` 不负责 workflow phase / gate / spec
- `drive` 只做一件事：按用户给的任务启动 detached codex
- 如果 codex 只是停下来问“是否继续”，就自动用 continue prompt 去 `resume`
- 监控只关心 4 类里程碑：
  - PR merge
  - CI 失败
  - 需要用户介入
  - drive 结束
- 非里程碑进展不通知用户，静默运行

## 执行方式

1. 把用户当前这次明确给出的任务，原样作为 drive prompt 主体
2. 用背景模式执行：

```bash
cc-leader drive "<user task>"
```

3. 这个命令会静默等待，直到出现一个里程碑再返回 JSON
4. 解析返回 JSON，至少读取：
   - `drive_id`
   - `active`
   - `stop_reason`
   - `milestone.type`
   - `milestone.summary`
   - `auto_continue_count`
   - `thread_id`
   - `stdout_log`
   - `stderr_log`
5. 里程碑处理：
   - `milestone.type == "pr_merged"`：通知用户 PR 已合并；如果 `active == true`，马上再次后台执行 `cc-leader drive` 挂下一个里程碑，不要问用户
   - `milestone.type == "ci_failed"`：通知用户 CI 失败；如果 codex 仍在继续修，`active == true` 时再次后台执行 `cc-leader drive`
   - `milestone.type == "needs_user"`：这才把控制权交回用户，展示 `milestone.summary`
   - `milestone.type == "drive_finished"`：通知用户 drive 已结束

## 接管已有 drive

如果上次 drive 启动后，`cc` 自己断线、报错、额度耗尽：

```bash
cc-leader drive
```

不带 prompt 时，表示接管当前 active drive；如果 active 已结束，也会回退读取最近一次 drive 的未通知里程碑。

如果当前 active drive 还在跑，`cc-leader drive` 会继续静默等下一个里程碑，而不是重复发新任务。

## 何时不用

- 需要 phase / review gate / 最终报告时，不用 `drive`，用 `/cc-leader-run`
- 需要先写 spec 时，不用 `drive`，用 `/cc-leader-spec`
