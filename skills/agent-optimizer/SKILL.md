---
name: agent-optimizer
description: "Optimize Clawdbot/OpenClaw agent operations: concurrency tuning, main↔subagent offloading, heartbeat/cron monitoring, and execution policy (auto-run vs confirmation) with safe risk tiers. Use when you want faster automation without sacrificing control for high-impact actions."
---

# Agent Optimizer / 代理优化器

> Goal: keep the **main agent responsive**, push **long-running work** to subagents, and adopt a **risk-tier execution policy** so routine automation does **not** wait for confirmation.

---

## 1) Execution policy (risk tiers) / 执行策略（风险分级）

### Tier A — Auto-run (no confirmation) / 自动执行（不确认）

Auto-run when the action is **read-only** or **reversible** and has **no external side effects**, e.g.:
- Read/query: files, status/list/get, web_fetch, sessions_list/history
- Workspace-only edits that are easy to revert (prefer small diffs; use git where available)
- Low-cost retries / idempotent checks

**Response pattern:** acknowledge + start immediately.
- CN: `收到，我开始做。预计 X 分钟，关键节点会回报。`
- EN: `Got it—starting now. ETA X min; I’ll report at milestones.`

### Tier B — Soft confirm (confirm only at the irreversible/high-cost step) / 轻确认（关键节点确认一次）

Use when:
- Meaningful cost/time (batch browser runs, many calls, heavy compute)
- A missing parameter gates correctness (scope, deadline, target)

**Approach:** do prep/dry-run first → ask for 1 confirmation at the “point of no return”.

### Tier C — Hard confirm (must confirm) / 强确认（必须确认）

Require explicit confirmation before actions with **external impact** or **hard-to-rollback** changes, e.g.:
- Send messages to third parties / broadcast
- Change gateway config / restart / self-update
- Deleting/overwriting important files, credential operations

### Stop & resume controls / 随时中止与继续

Treat these as high priority:
- `stop/暂停` → stop further tool calls / ask subagents to stop
- `continue/继续` → resume from last safe checkpoint

---

## 2) Offload policy (main ↔ subagents) / 主/子代理卸载策略

### Main agent responsibilities / 主代理职责
- Conversation + clarification
- Scheduling / orchestration
- Summaries + user-facing reporting
- Monitoring subagents

### Offload triggers / 触发卸载条件（满足任一条就丢给子代理）
- Expected runtime > 60–120s
- Multi-step browser automation
- ≥2 independent subtasks that can run in parallel
- Anything likely to block interactive chat

**Subagent spawning (example) / 子代理调用（示例）**
```js
await sessions_spawn({
  task: "Describe the long-running task clearly",
  label: "short-label-for-status",
  cleanup: "delete"
});
```

---

## 3) Concurrency tuning / 并发调优

### Baseline recommendations / 基线建议
- Main `maxConcurrent`: **8–12** (optimize for responsiveness)
- Subagents `maxConcurrent`: **12–20** (optimize for throughput)

### Adaptive guidance / 自适应建议
- If latency/timeout rises → reduce subagent concurrency first.
- If chat feels sluggish → reduce main concurrency.

**Apply via config.patch / 配置示例**
```bash
clawdbot gateway config.patch --raw '{"agents":{"defaults":{"maxConcurrent":10,"subagents":{"maxConcurrent":16}}}}'
```

---

## 4) Monitoring: heartbeat + cron (don’t spam) / 监控：心跳 + 定时（避免刷屏）

### Default behavior / 默认行为
- Check every 10 minutes, but **only notify on triggers**.

### Notification triggers / 发送汇报的触发条件
- Any task failed
- Any task running > 30 min (configurable)
- State transitions: running tasks 0→>0 or >0→0
- User explicitly enabled “periodic report mode”

### Report format / 汇报格式
```
🔄 Task Status / 任务状态
- Running / 运行中: X
  - [label] age=12m status=running last="..."
- Recent done / 近期完成: Y
  - [label] dur=3m result=success
  - [label] dur=1m result=failed reason="..."
```

---

## 5) Resources / 资源

- `references/heartbeat-template.md`
- `references/memory-template.md`

Use templates as a starting point; customize thresholds and reporting verbosity.
