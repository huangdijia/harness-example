# 规格说明: minimal-todo

## 状态

- workflow_id: wf-minimal-todo-20260412-ab12
- status: approved
- owner: cc
- latest_review: docs/examples/minimal-todo/spec-review.md

## 目标

给一个极小 todo CLI 增加新增和列表能力。

## 非目标

- 不做多用户
- 不做网络同步

## 约束

- 改动保持在单仓库内
- 必须可命令行验证

## 成功标准

- `todo add "milk"` 可写入数据
- `todo list` 可读出数据

## 选定方案

先实现本地文件存储，再实现命令分支。

## 架构与边界

- CLI 入口负责命令分发
- 存储模块负责读写 JSON 文件

## Phase 假设

- 这项工作足够小，只需 1 个 phase

## 测试与验证预期

- 用 smoke 命令验证 add/list

## 开放问题

- none

## 审查历史

- 2026-04-12: 初稿完成并通过对抗式 review

## 批准

- spec_review_verdict: pass
- user_approved: yes

## Override 记录

- none
