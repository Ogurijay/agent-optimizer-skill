# HEARTBEAT.md / 心跳任务模板

## Goal / 目标
Check subagent task health periodically **without spamming**. Prefer **trigger-based notifications**.

每隔一段时间检查子代理任务状态，但**只在需要时汇报**，避免刷屏。

---

## Schedule / 建议频率
- Check interval: every **10 minutes** (or longer if you prefer)
- 通常 `*/10 * * * *`

---

## Steps / 执行步骤

1) **List sessions / 获取会话列表**
- Use `sessions_list` (limit ~50)
- Exclude main session

2) **Classify / 分类**
- Running: active subagent sessions
- Done: recently finished sessions (if available) or sessions with final status message

3) **Compute age / 计算运行时长**
- `now - createdAt` (or infer from first message timestamp)

4) **Decide whether to notify / 是否需要发消息（触发器）**
Notify only if any of these is true:
- Any failure detected
- Any task age > 30 minutes (configurable)
- State transition: running 0→>0 or >0→0
- User enabled periodic report mode

5) **Send report / 发送汇报**
Use `message` tool to send a compact report.

---

## Report format / 汇报格式
```
🔄 Task Status / 任务状态
- Running / 运行中: X
  - [label] age=12m status=running last="..."
- Recent done / 近期完成: Y
  - [label] dur=3m result=success
  - [label] dur=1m result=failed reason="..."
```

## Notes / 注意
- If nothing notable happened → send nothing (or a one-liner only if user asked).
- Focus on actionable info: label, age, last output snippet, failure reason.
