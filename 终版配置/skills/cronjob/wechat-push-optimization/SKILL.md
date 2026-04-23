---
name: wechat-push-optimization
category: cronjob
description: Strategy for optimizing WeChat message推送 to avoid timeouts and gateway restarts during long cron job executions
---

# WeChat Push Optimization for Cron Jobs

## Problem
WeChat推送失败 due to:
1. **Message too long** (>3000 chars) → Timeout errors
2. **Gateway restarts** during execution → Task interruption
3. **Single long push** → High risk of failure

## Solution: Multi-Step Serial Pushing

### Core Strategy
- **Split content** into 4 sequential sub-tasks
- **Each push ≤ 3000 chars**
- **Serial execution**: Sub-task 1 → 2 → 3 → 4 (auto-triggered)
- **Total duration**: 2-3 minutes

### Implementation Pattern

```yaml
# Task prompt structure
每日 22:00 执行分析，**严格串行执行 4 个子任务**：

### 子任务 1：新闻抓取
- 国际新闻 (15 条) + 中国新闻 (10 条)
- 每条：标题 + 摘要 (80-120 字) + 影响评级 + 解读

### 子任务 2：深度分析
- 5 视角分析 (政治家/经济学家/历史学家/社会学家/企业家)
- 每视角 150-250 字

### 子任务 3：市场预测
- 港股 + 美股预测 (乐观/中性/悲观情景)
- 置信度标注 + 驱动因素

### 子任务 4：配置建议
- 行业轮动 + 对冲策略 + 风险提示
- 总结 + 行动建议

✅ 每个子任务完成后自动触发下一个
```

### Content Limits
| 部分 | 字数限制 | 内容要求 |
|------|---------|---------|
| **单条推送** | ≤ 3000 字 | 确保 WeChat 稳定 |
| **新闻条目** | 150-200 字/条 | 标题 + 摘要 + 评级 + 解读 |
| **分析视角** | 150-250 字/视角 | 深度分析 |
| **总执行时间** | 2-3 分钟 | 避免网关重启 |

### Key Rules
1. **Serial execution only**: No parallel pushes
2. **Auto-trigger**: Each sub-task triggers next automatically
3. **Content balance**: Complete analysis but within limits
4. **Bullet points**: Avoid long paragraphs
5. **Charts optional**: Max 1 chart if necessary

### Debugging Tips
If推送失败:
1. Check gateway logs: `tail -100 /root/.hermes/logs/agent.log`
2. Look for: `Weixin send failed`, `Timeout context manager`
3. Verify: `cronjob list` shows `last_status: ok`
4. Reduce content if still failing

### Example Workflow
```bash
# Create task with optimized prompt
cronjob create \
  --name "Global Market Analysis" \
  --prompt "Daily 22:00 analysis with 4 serial sub-tasks..." \
  --schedule "0 14 * * *" \
  --skills macro-policy-impact-analysis,multi-perspective-macro-analysis

# Test manually
cronjob run --job_id <job_id>

# Monitor execution
tail -f /root/.hermes/logs/agent.log | grep cron_<job_id>
```

## Common Pitfalls
- ❌ **Too many sub-tasks**: Keep to 4 max
- ❌ **Parallel pushes**: Always use serial
- ❌ **Ignoring content limits**: Stay ≤ 3000 chars/push
- ❌ **Gateway restarts**: Shorter execution = more stable