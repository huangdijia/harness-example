---
name: reviewing-phase-results
description: 当某个 phase 执行完成后，worker 需对照 plan 做 phase review；全部 phase 通过后，再做 spec 级 final review 时使用。
---

# 审查阶段结果

每个 phase 执行完后，用这个 phase 审结果。全部 phase 都过后，还要再做一次 spec 级 final review。

## 角色

- `cc`: 决定审哪些 artifact；如果失败，决定回到 tasking 还是 execution；如果通过，推进到下一 phase
- `worker`: 对照 planning artifact 和 result log 做深审

## 规则

- 每个已完成 phase 都要先 review，再开下一个依赖 phase
- review 要对照 phase plan，不只看 diff
- 全部 phase 过后，还要对完整 spec 再审一次
- review 失败时，只重开最小必要范围

## 流程

### 阶段审查

1. 读 phase plan、phase task list、phase result
2. 派 worker 做完整 phase review
3. 要求 worker 检查：
   - 所有计划 deliverable 都存在
   - task result 足以支撑宣称结果
   - verification 证据够
   - phase 没偏离原 goal
   - follow-up work 明确
4. 把 phase review 输出落盘，verdict 为 `pass` 或 `fail`
5. 如果 review 失败：
   - 按问题类型回到 task generation 或 execution
6. 如果 review 通过，且下一个 phase 已 ready：
   - 回到 `executing-phase-tasks`

### 最终规格审查

7. 全部 phase review 都通过后，派 worker 对照原始 spec 做最终 review
8. 保存 final spec review
9. 如果 final spec review 失败：
   - 回到受影响最小 phase
10. 如果 final spec review 通过：
   - 进入 `reporting-results`

## 审查输出必须包含

- verdict
- evidence reviewed
- findings
- required follow-ups，或明确 `none`
- 对齐目标 artifact 的结论

## 退出条件

只有两种情况能离开：

- 当前 phase review 已通过，workflow 可进下一步
- final spec review 已通过，workflow 可进入 reporting

## 反模式

- 只看代码，不看 plan/spec
- 某个 phase review 失败还偷偷往后滚
- 跳过 final spec-level review
