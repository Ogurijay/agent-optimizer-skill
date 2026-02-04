---
name: agent-optimizer
description: "Configure Clawdbot agent settings for concurrency, heartbeat monitoring, and task execution. Use to adjust agent/subagent limits, set up monitoring, offload long-running tasks, and establish reporting."
---

# Agent Optimizer

## Overview

This skill configures Clawdbot agent settings to optimize performance and responsiveness. It handles concurrency tuning, heartbeat monitoring setup, and establishes best practices for main agent vs subagent task distribution.

## Core Capabilities

### 1. Concurrency Configuration

Adjust how many tasks the main agent and subagents can run concurrently.

**Recommended Settings:**
- Main agent `maxConcurrent`: 10-12 (for interactive responsiveness)
- Subagent `maxConcurrent`: 16-20 (for background task throughput)

**How to Apply:**
```bash
clawdbot gateway config.patch --raw '{"agents":{"defaults":{"maxConcurrent":10,"subagents":{"maxConcurrent":16}}}}'
```

### 2. Heartbeat Monitoring Setup

Create automated monitoring for subagent tasks with periodic status reports.

**Implementation:**
- Create a cron job with `*/10 * * * *` schedule (every 10 minutes)
- Use `--system-event` trigger to run in main session context
- Define monitoring logic in workspace `HEARTBEAT.md`

**Example:**
```bash
clawdbot cron add \
  --name "task-heartbeat" \
  --cron "*/10 * * * *" \
  --system-event "检查子代理任务状态并汇报" \
  --session main
```

### 3. Main Agent Behavior Guidelines

**Main Agent Responsibilities:**
- User conversation and interaction
- Message receipt and acknowledgment
- Task scheduling and orchestration
- Subagent status monitoring

**Main Agent Must NOT Handle (offload to subagents):**
- Image generation (image-generate)
- Video generation (video-generate)
- Browser automation (playwright-skill)
- TikTok KOL finding (tiktok-kol-finder)
- AI news tracking (ai-news-tracker)
- Any other long-running or blocking tasks

**Using Subagents:**
```python
# Spawn subagent for long-running tasks
await sessions_spawn({
  task: "具体的任务描述",
  label: "任务标签",
  agentId: "main",
  cleanup: "delete"
});
```

### 4. Message Handling Workflow

**When Receiving Task Requests:**

1. **Immediate Acknowledgment**
   - Reply: `收到。` (first response, immediate)
   - Do not make any tool calls before this acknowledgment

2. **Plan and Present**
   - Analyze the request
   - Create an execution plan
   - Present the plan to the user

3. **Wait for Confirmation**
   - Do NOT start execution
   - Wait for explicit user confirmation

4. **Execute After Confirmation**
   - Only begin tool calls and task execution after user confirms

**Example Flow:**
```
User: Push this skill to GitHub
Agent: 收到。

Agent: Here's my execution plan:
       1. Initialize git repo
       2. Add skill file
       3. Create README
       4. Commit and push

User: Confirmed / OK
Agent: [Starts execution with git commands]
```

**Important Rules:**
- Always reply `收到。` BEFORE any tool calls
- Never start execution before user confirms the plan
- This ensures user has full control over task execution
- The plan should include: what will be done, how, and any required inputs

### 5. Task Reporting Standards

**When receiving task execution confirmation:**
- User has reviewed and approved the plan
- Immediately spawn subagent or execute the task

**When task completes (success or failure):**
- Success: `✅ 任务完成：[任务描述]`
- Failure: `❌ 任务失败：[任务描述] - [错误原因]`

**Heartbeat Report Format:**
```
🔄 任务状态汇报
- 运行中：X 个任务
  - [任务1] 类型：xxx，时长：X分X秒，状态：运行中
  - [任务2] 类型：xxx，时长：X分X秒，状态：xxx
- 近期完成：Y 个任务
  - [任务1] 类型：xxx，耗时：X分X秒，结果：成功
  - [任务2] 类型：xxx，耗时：X分X秒，结果：失败
```

## Workflow: Complete Agent Optimization

### Step 1: Apply Concurrency Settings

Use `gateway config.patch` to set main and subagent concurrency limits.

### Step 2: Create Heartbeat Monitor

Use `clawdbot cron add` to create a periodic monitoring job.

### Step 3: Define Heartbeat Logic

Create `HEARTBEAT.md` in workspace with monitoring workflow instructions.

### Step 4: Document Behavior Guidelines

Update `MEMORY.md` with main agent responsibilities and task offloading rules.

### Step 5: Restart Gateway (if needed)

Gateway restarts automatically after config changes via SIGUSR1.

## Resources

### scripts/

- No scripts currently needed. Configuration is applied via gateway and cron tools.

### references/

- `memory-template.md` - Template for MEMORY.md with agent behavior guidelines
- `heartbeat-template.md` - Template for HEARTBEAT.md with monitoring workflow

## Validation Checklist

Before considering the optimization complete, verify:

- [ ] Concurrency settings applied correctly (`clawdbot gateway config.get`)
- [ ] Cron job created and scheduled (`clawdbot cron list`)
- [ ] HEARTBEAT.md exists with monitoring instructions
- [ ] MEMORY.md documents agent behavior and task offloading
- [ ] Gateway restarted and new settings active

## Common Issues

**Issue:** Cron job not appearing in list
- **Solution:** Verify job creation command output, check cron status with `clawdbot cron status`

**Issue:** Concurrency settings not taking effect
- **Solution:** Gateway should auto-restart via SIGUSR1. If not, check logs and manual restart

**Issue:** Heartbeat not firing
- **Solution:** Verify `HEARTBEAT.md` exists and cron job `payload.kind` is "systemEvent"
