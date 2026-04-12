# CC Leader

给 Claude Code 用的多 agent workflow skill pack 骨架。

核心约束：

- `cc` 面向用户，负责状态推进
- `worker` 目前只支持 `codex-cli`
- `worker` 负责生成、执行、审查
- 关键阶段都要落盘
- 前进要过 review gate

## 阶段总览

1. `drafting-specs`
   - 和用户写 spec
   - `worker` 做对抗式 review
2. `writing-phase-plans`
   - 基于已批准 spec 写 phase plan list
   - 先 self review
3. `writing-phase-tasks`
   - 按 phase 并发生成 phase task list
   - 先 self review
4. `executing-phase-tasks`
   - 按依赖顺序执行 task list
   - 逐 task 追加 phase result
5. `reviewing-phase-results`
   - 做 phase review
   - 全部完成后做 final spec review
6. `reporting-results`
   - `cc` 汇总最终报告

## 关键入口

- manifest: `cc-leader.manifest.json`
- bootstrap skill: `skills/using-cc-leader/SKILL.md`
- loader 合同: [docs/loader-contract.md](docs/loader-contract.md)
- transport 合同: [docs/transport-contract.md](docs/transport-contract.md)
- runtime 合同: [docs/runtime-contract.md](docs/runtime-contract.md)
- 状态机: [docs/state-machine.md](docs/state-machine.md)
- 产物规则: [docs/artifact-rules.md](docs/artifact-rules.md)
- verification 合同: [docs/verification-contract.md](docs/verification-contract.md)
- 编排合同: [docs/worker-orchestration-contract.md](docs/worker-orchestration-contract.md)
- prompt 合同: [docs/worker-prompt-contracts.md](docs/worker-prompt-contracts.md)
- harness 样例: [docs/harness-example.md](docs/harness-example.md)
- 最小 harness 脚本: `scripts/cc-leader-harness.mjs`
- 一键 smoke 脚本: `scripts/smoke-run.mjs`
- artifact 样例: `docs/examples/minimal-todo/`
- dispatch prompt: `docs/prompts/`
- artifact 模板: `docs/templates/`

## 状态

这是 workflow 骨架，不是完整 runtime。

已定义：

- phase skills
- session state key
- artifact 目录与命名规则
- worker runtime: `codex-cli`
- transport/runtime 合同
- worker prompt 合同
- 状态机、重试、override、verification 规则
- 最小 harness 脚本
- `report` 命令
- 样例 artifact
- 自检脚本

未定义：

- 生产级 harness 实现
- 调度服务实现
