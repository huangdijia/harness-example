# CC Leader Verification 合同

这个文档定义 verification 责任和最低标准。

## 责任分配

- phase plan worker：为每个 phase 写 verification strategy
- phase task worker：为每个 task 写具体 verification
- phase execution worker：真的跑 verification
- phase review worker：复核 evidence，必要时重跑快速验证
- final report：引用 verification evidence

## 三层验证

### 1. Task 级

每个 task 都必须有至少一条 verification：

- 命令
- 或明确人工检查步骤

执行 worker 必须记录：

- 跑了什么
- pass/fail
- 关键输出摘要

### 2. Phase 级

phase review 至少要做其一：

- 重跑 1 条快速验证
- 或严格核对 execution log 中的验证证据

如果 evidence 不足，phase review 不能给 `pass`。

### 3. Final 级

final spec review 必须检查：

- spec success criteria 是否有证据
- 是否还有未覆盖 requirement

## 命令分类

- smoke: 很快，适合 phase review 复验
- focused: 针对单 task/单模块
- full: 完整验收

phase task list 至少给 smoke/focused。final report 应引用 full 或完整组合证据。

## 失败规则

- 同一 verification 错误重试 2 次还失败 -> `blocked`
- 环境缺失导致无法验证 -> `blocked`
- 不允许静默跳过 verification

## 记录格式

artifact 里至少记录：

- verification 命令或检查项
- pass/fail
- 关键摘要

详细日志可放 stdout/stderr log，不必全文塞进 markdown。
