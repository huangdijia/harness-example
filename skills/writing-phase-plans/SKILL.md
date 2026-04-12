---
name: writing-phase-plans
description: 当已批准 spec 存在，且需要 worker 把它转成多 phase plan list 并 self review 时使用。
---

# 编写阶段计划

spec 批准后、task 拆分前，用这个 phase。

## 角色

- `cc`: 检查是否 ready，派 planning worker，校验返回的 phase list 是否对齐 spec
- `worker`: 写 phase plan list，并 self review

## 规则

- 这个 phase 里不要开始 task generation
- 每个 phase 都要是有意义、可验证的增量
- 依赖必须写明
- 每个 phase 都要小到能独立 task、独立 review

## 流程

1. 读已批准 spec
2. 识别为了安全达成 spec 目标需要哪些 phase 边界
3. 派 worker 生成 phase plan list
4. 要求 worker 返回前先 self review 整个 phase list
5. self review 必查：
   - 每个 spec requirement 都有覆盖
   - phase 顺序有效
   - dependencies 明确
   - 每个 phase 的 deliverable 和 verification 明确
   - 没有把大块未知工作藏进 phase
6. 把经过 review 的 phase plan list 落盘
7. `cc` 做一次快速 coordinator check：
   - 列表内部一致
   - phase id 稳定
   - 后续 per-phase task generation 不需要猜
8. 准备好后，进入 `writing-phase-tasks`

## 每个阶段必含字段

- phase id
- title
- goal
- depends on
- deliverables
- likely files or modules
- verification strategy
- risks 和 rollback notes

## 退出条件

只有全部满足才离开：

- phase plan list 已落盘
- worker self-review 认定 plan 对齐 spec
- `cc` 确认可进入 per-phase task generation

## 反模式

- 一个巨型 phase 吞掉全部工作
- phase 没依赖信息
- verification 全拖到最后
