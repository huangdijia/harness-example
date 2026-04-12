# CC Leader Harness 样例

这个文档给一个最小可跑 harness 思路。

## 目标

做这些事：

1. 读 manifest
2. 决定当前 phase
3. 渲染 worker prompt
4. 调 `codex exec`
5. 读结果文件
6. 更新 session state

仓库里已有最小脚本：

- `scripts/cc-leader-harness.mjs`
- `scripts/smoke-run.mjs`
- `npm run harness -- <command>`
- `npm run smoke`

## 最小命令

```bash
npm run harness -- init --slug todo-cli
npm run harness -- state:get
npm run harness -- resolve-phase
npm run harness -- prepare-job --job specAdversarialReview
npm run harness -- dispatch --job specAdversarialReview --timeout-seconds 120
npm run harness -- report --final-report-path docs/reports/example.md
```

## 目录准备

```bash
mkdir -p ".cc-leader/runs/$WORKFLOW_ID/$JOB_ID"
mkdir -p "$(dirname "$RESULT_FILE")"
```

## 最小 shell 流程

```bash
WORKFLOW_ID="wf-todo-cli-20260412-ab12"
JOB_ID="job-phase-review-phase-01-core-todo-20260412t100000z"
RUN_DIR=".cc-leader/runs/$WORKFLOW_ID/$JOB_ID"
PROMPT_FILE="$RUN_DIR/prompt.md"
STDOUT_LOG="$RUN_DIR/stdout.jsonl"
STDERR_LOG="$RUN_DIR/stderr.log"
RESULT_FILE="$RUN_DIR/result.json"

mkdir -p "$RUN_DIR"

# 1. 渲染 prompt 到 $PROMPT_FILE
# 2. 把 result_file_path 变量填成 $RESULT_FILE

PROMPT_CONTENT="$(cat "$PROMPT_FILE")"
codex exec --json --dangerously-bypass-approvals-and-sandbox "$PROMPT_CONTENT" \
  < /dev/null \
  > "$STDOUT_LOG" \
  2> "$STDERR_LOG"

# 3. 进程退出后，读 $RESULT_FILE
```

## 最小 TypeScript 伪代码

```ts
const manifest = readJson("cc-leader.manifest.json");
const state = loadState();
const phase = resolvePhase({ state, manifest, userMessage });

const job = buildJob({
  manifest,
  phase,
  workflowId: ensureWorkflowId(state),
});

ensureDir(job.runDir);
ensureDir(dirname(job.resultFile));
ensureDir(dirname(job.outputArtifact));

renderPromptToFile(job.promptFile, {
  ...job.variables,
  result_file_path: job.resultFile,
});

const exitCode = runCodex(job);
const result = readJson(job.resultFile);
assertArtifactsExist(result.outputs);
state = reduceState({ state, result, exitCode, manifest });
saveState(state);
```

## 关键点

- 不要从 stdout 抓业务状态
- 只认 `result.json`
- 每次重试都用新 `job_id`
- 写 artifact 前先 `mkdir -p`
- code-changing phase 开始前打 rollback anchor

## 参考

- `docs/runtime-contract.md`
- `docs/state-machine.md`
- `docs/transport-contract.md`
