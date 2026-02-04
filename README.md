# Agent Optimizer Skill

[English](#english) | [中文](#chinese)

---

## Chinese / 中文

### 简介

Agent Optimizer Skill 是用于优化 Clawdbot 代理配置和行为的专用技能。它提供了并发调优、心跳监控设置、主/子代理任务分配等完整的系统优化方案。

### 功能特性

- **并发配置优化**：调整主代理和子代理的并发任务限制
- **心跳监控设置**：自动化的子代理任务状态监控和汇报
- **任务分配策略**：明确主代理与子代理的职责划分
- **消息处理规范**：标准化的任务请求和响应流程
- **状态报告标准**：统一的任务状态汇报格式

### 安装方法

1. 下载 `.skill` 文件
2. 将其放置到你的 Clawdbot 技能目录中
3. 重启 Clawdbot 或重新加载技能

### 使用指南

#### 1. 配置并发设置

```bash
clawdbot gateway config.patch --raw '{
  "agents": {
    "defaults": {
      "maxConcurrent": 10,
      "subagents": {
        "maxConcurrent": 16
      }
    }
  }
}'
```

#### 2. 创建心跳监控任务

```bash
clawdbot cron add \
  --name "task-heartbeat" \
  --cron "*/10 * * * *" \
  --system-event "检查子代理任务状态并汇报" \
  --session main
```

#### 3. 定义监控逻辑

在工作区创建 `HEARTBEAT.md` 文件，定义监控流程和汇报格式。

#### 4. 更新行为规范

在工作区创建或更新 `MEMORY.md`，记录主代理职责和任务卸载规则。

### 配置说明

#### 主代理职责

- 用户对话和交互
- 消息接收和确认
- 任务调度和编排
- 子代理状态监控

#### 子代理负责的任务

- 图片生成（image-generate）
- 视频生成（video-generate）
- 浏览器自动化（playwright-skill）
- TikTok KOL 查找（tiktok-kol-finder）
- AI 新闻追踪（ai-news-tracker）
- 其他任何长时间运行或阻塞的任务

#### 消息处理流程

1. 收到任务请求 → 回复 `收到。`
2. 制定执行计划并汇报
3. **等待用户确认**
4. 收到确认后才开始执行任务

### 开源协议

MIT License - 使用此技能时需要保留原作者声明。

---

## English

### Overview

Agent Optimizer Skill is a specialized skill for optimizing Clawdbot agent configuration and behavior. It provides a comprehensive system optimization solution including concurrency tuning, heartbeat monitoring setup, and main/subagent task distribution.

### Features

- **Concurrency Configuration**: Adjust concurrent task limits for main agent and subagents
- **Heartbeat Monitoring**: Automated subagent task status monitoring and reporting
- **Task Distribution Strategy**: Clear division of responsibilities between main agent and subagents
- **Message Handling Standards**: Standardized task request and response workflows
- **Status Reporting**: Unified task status reporting format

### Installation

1. Download the `.skill` file
2. Place it in your Clawdbot skills directory
3. Restart Clawdbot or reload skills

### Usage Guide

#### 1. Configure Concurrency Settings

```bash
clawdbot gateway config.patch --raw '{
  "agents": {
    "defaults": {
      "maxConcurrent": 10,
      "subagents": {
        "maxConcurrent": 16
      }
    }
  }
}'
```

#### 2. Create Heartbeat Monitoring Job

```bash
clawdbot cron add \
  --name "task-heartbeat" \
  --cron "*/10 * * * *" \
  --system-event "Check subagent task status and report" \
  --session main
```

#### 3. Define Monitoring Logic

Create `HEARTBEAT.md` in your workspace to define monitoring workflow and report format.

#### 4. Update Behavior Guidelines

Create or update `MEMORY.md` in your workspace to document main agent responsibilities and task offloading rules.

### Configuration

#### Main Agent Responsibilities

- User conversation and interaction
- Message receipt and acknowledgment
- Task scheduling and orchestration
- Subagent status monitoring

#### Subagent-Handled Tasks

- Image generation (image-generate)
- Video generation (video-generate)
- Browser automation (playwright-skill)
- TikTok KOL finding (tiktok-kol-finder)
- AI news tracking (ai-news-tracker)
- Any other long-running or blocking tasks

#### Message Handling Workflow

1. Receive task request → Reply: `收到。` (Received)
2. Create execution plan and present to user
3. **Wait for user confirmation**
4. Only start execution after receiving explicit confirmation

### License

MIT License - Original attribution must be retained when using this skill.

---

**Copyright © 2026 小G (XiaoG)**
