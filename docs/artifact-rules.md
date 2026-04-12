# CC Leader 产物规则

这个文档定义 artifact 命名、路径、目录创建、schema 最低要求。

## 路径规则

业务 artifact 默认放在 repo 内：

- spec: `docs/specs/<workflow-id>.md`
- spec review: `docs/reviews/specs/<workflow-id>.md`
- override log: `docs/reviews/overrides/<workflow-id>.md`
- phase plan list: `docs/plans/<workflow-id>.md`
- phase task list: `docs/tasks/<phase-id>.md`
- phase result: `docs/results/<phase-id>.md`
- phase review: `docs/reviews/phases/<phase-id>.md`
- final spec review: `docs/reviews/spec-final/<workflow-id>.md`
- final report: `docs/reports/<workflow-id>.md`

运行时文件放在：

- `.cc-leader/runs/<workflow-id>/<job-id>/`

## 目录创建

仓库不预置空目录。

写任何 artifact 前，`cc` 或 worker 必须：

```bash
mkdir -p "$(dirname "$TARGET_PATH")"
```

## ID 规则

- `workflow_id`: `wf-<slug>-<YYYYMMDD>-<4hex>`
- `phase_id`: `phase-<NN>-<slug>`
- `task_id`: `task-<NN>-<slug>`
- `job_id`: `job-<job-type>-<global-or-phase-id>-<YYYYMMDDtHHMMSSz>`

这些格式是硬约束，不要自由命名。

## 路径格式

- markdown artifact 里的文件路径：优先用 repo 相对路径
- result file 里的输出路径：必须用绝对路径

## Schema 最低要求

每类 artifact 必须：

- 使用对应模板
- 保留模板里的固定标题
- 必填字段不能缺
- 不得残留 `<placeholder>` 或 `TBD`

如果缺字段，artifact 视为无效。

## Override 记录

发生 override 时：

- spec、phase review、final spec review、final report 必须写 `Override` 相关段
- 同时追加到 override log
- 没 override 时也要写 `none`

## 产物有效性检查

最少检查：

- id 格式合法
- 路径和 phase 对应关系正确
- verdict/status 在允许枚举里
- 模板标题完整
- 无占位符残留

## 样例

参考：

- `docs/examples/minimal-todo/`
