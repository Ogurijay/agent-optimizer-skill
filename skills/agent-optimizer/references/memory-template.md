# MEMORY.md template / 长期记忆模板

> Customize this for your deployment. Keep it short and operational.

---

## Execution Policy / 执行策略（默认）

### Tier A — Auto-run / 自动执行（不确认）
- Read-only queries, status/list/get
- Workspace-only reversible edits
- Low-cost checks

### Tier B — Soft confirm / 轻确认
- Confirm only at irreversible/high-cost step

### Tier C — Hard confirm / 强确认
- External side effects (messaging others)
- Gateway config/restart/self-update
- Deletes/overwrites of important files

**Stop command / 中止口令:** `stop/暂停`

---

## Main vs Subagents / 主代理与子代理

### Main agent does / 主代理做
- Conversation + clarifying questions
- Scheduling/orchestration
- Summaries + reporting
- Monitoring

### Offload triggers / 卸载触发器
- >60–120s expected runtime
- Browser automation
- Parallelizable work
- Anything that may block chat

---

## Concurrency baseline / 并发基线
- Main `maxConcurrent`: 8–12
- Subagents `maxConcurrent`: 12–20

Adaptive rule / 自适应规则:
- Latency↑/failures↑ → reduce subagent concurrency first
- Chat sluggish → reduce main concurrency

---

## Monitoring / 监控
- Check every ~10 minutes
- Notify on triggers only (failure, long-running, state transitions)
