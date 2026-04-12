---
name: writing-phase-tasks
description: 当 cc 已有 phase plan list，且需要并发派 worker 为每个 phase 生成并 self review task list 时使用。
---

# 编写阶段任务

phase plan list 已有，要产出可执行 task list 时，用这个 phase。

## 角色

- `cc`: 读取 phase plan list，并发派 worker，追踪 ready 的 task list
- `worker`: 独占一个 phase plan，把它扩成 task list，再 self review

## 规则

- 一个 worker 一次只负责一个 phase plan
- task generation 可以跨 phase 并发
- 每个 task 都必须可执行，不能靠隐含上下文
- 后续 phase 可提前准备，但执行仍要守依赖

## 流程

1. 读 phase plan list
2. 对每个 phase plan 派一个 worker 生成 phase task list
3. 要求 worker：
   - 重述 phase goal
   - 产出有序 task list
   - 对 task list 做 self review
4. self review 必须确认：
   - phase plan 里的 deliverable 都被覆盖
   - task 顺序有效
   - 每个 task 都有 verification
   - blocker 和 assumption 明确
5. 每个 phase task list 按稳定 phase id 落盘
6. 只有 task list 和 self review 都完成，phase 才能标记 ready
7. 只要第一个可执行 phase task list ready，`cc` 就可以进入 `executing-phase-tasks`

## 每个任务必含字段

- task id
- objective
- files 或 artifacts touched
- 有序 implementation steps
- verification commands 或 review checks
- completion criteria
- rollback 或 retry note（如有）

## 协调者检查

phase 标 ready 前，`cc` 应确认：

- phase id 和 phase plan list 一致
- task list 自包含
- dependency assumption 已写出

## 退出条件

满足下面就可离开：

- 至少一个可执行 phase task list 已 ready
- 其他 phase task list 要么也 ready，要么仍在并发生成

## 反模式

- 一个 worker 同时改多个无关 phase task list
- 模糊 task，比如 “handle edge cases”
- 没 verification 的 task list
