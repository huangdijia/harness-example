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
- 只有真实 blocker、真正缺输入、或需要用户业务决策时，才把控制权交回用户

## 执行方式

1. 把用户当前这次明确给出的任务，原样作为 drive prompt 主体
2. 用背景模式执行：

```bash
cc-leader drive "<user task>"
```

3. 解析返回 JSON，至少读取：
   - `drive_id`
   - `status`
   - `stop_reason`
   - `auto_continue_count`
   - `thread_id`
   - `latest_message`
   - `stdout_log`
   - `stderr_log`
4. 结果处理：
   - `stop_reason == "completed"`：向用户汇报已完成
   - `stop_reason == "waiting_for_user"`：展示 codex 最后消息；只有这时再由 `cc` 判断是否重进 `cc-leader drive`
   - `stop_reason == "transport_failed"`：展示错误和日志路径

## 接管已有 drive

如果上次 drive 启动后，`cc` 自己断线、报错、额度耗尽：

```bash
cc-leader drive
```

不带 prompt 时，表示接管当前 active drive。

## 何时不用

- 需要 phase / review gate / 最终报告时，不用 `drive`，用 `/cc-leader-run`
- 需要先写 spec 时，不用 `drive`，用 `/cc-leader-spec`
