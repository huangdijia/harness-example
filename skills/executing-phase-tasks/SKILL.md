---
name: executing-phase-tasks
description: 当已审过的 phase task list ready，且 worker 要逐 task 执行并在每步后追加结果时使用。
---

# 执行阶段任务

当 `cc` 已有 ready 的 phase task list，要开始执行时，用这个 phase。

## 角色

- `cc`: 选下一个可执行 phase，派 worker，追踪进度和 blocker
- `worker`: 按顺序执行一个 phase task list，并在每个 task 后追加结果

## 规则

- 同一时间只执行一个活跃依赖 phase
- 除非 task list 明确允许，否则不要跳 task
- 每完成一个 task，就追加 phase result
- 如果 verification 失败，或 task list 和仓库现实不符，就停并上报

## 流程

1. 选下一个可执行 phase：
   - task list 已 ready
   - dependencies 已满足
   - 前置必需 phase review 已通过
2. 派 worker 执行该 phase task list
3. 要求 worker 按顺序处理 task
4. 每个 task 后，在 phase result 文档追加：
   - task id
   - status
   - summary of work done
   - changed files 或 artifacts
   - verification run
   - follow-up notes
5. 如果 task 阻塞：
   - 在 phase result 文档里记录 blocker
   - 立即通知 `cc`
   - 停止，不要猜
6. 最后一个 task 完成后，标记 phase execution complete，并进入 `reviewing-phase-results`

## 执行纪律

- 一次只做一个 task
- 每个 task 都要显式 verification
- result history 只追加
- 执行中不要偷偷改 plan

## 退出条件

只有两种情况能离开：

- 当前 phase task list 已全部执行完
- 执行阻塞，需要 `cc` 介入

## 反模式

- 多个 task 打包成一个未检查的大改动
- 丢失逐 task 执行历史
- phase result 还没更新完就宣称完成
