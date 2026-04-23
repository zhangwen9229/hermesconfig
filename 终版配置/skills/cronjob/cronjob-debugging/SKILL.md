---
name: cronjob-debugging
category: cronjob
description: Systematic approach to diagnose and fix cron job execution failures, especially WeChat推送 issues
---

# Cron Job Debugging Guide

## Problem Symptoms

### 1. "Skill(s) not found and skipped"
**Cause**: Configured tools as skills instead of actual skills

**Diagnosis**:
```bash
# Check if skill exists
ls /root/.hermes/skills/<skill_name>
# Returns: No such file or directory
```

**Solution**:
- ✅ Only list actual skills (directories in `/root/.hermes/skills/`)
- ❌ Don't list tools: `web-search`, `send-message`, `file`, `terminal`
- Tools are automatically available in task context

### 2. WeChat推送失败 / Timeout
**Symptoms**:
- `Weixin send failed: Timeout context manager`
- `iLink sendmessage error: ret=-2`
- Gateway restarts during execution

**Diagnosis**:
```bash
# Check task status
cronjob list

# Look for errors
tail -100 /root/.hermes/logs/agent.log | grep -E "(cron_<job_id>|Weixin|send)"

# Check if gateway restarted
tail -100 /root/.hermes/logs/agent.log | grep -E "(restart|stop|Gateway)"
```

**Root Causes**:
1. Message too long (>3000 chars)
2. Gateway memory issues
3. Network instability
4. Execution time too long (>3 minutes)

### 3. Task Not Executing
**Symptoms**:
- `last_run_at: null`
- `last_status: null`
- No logs for job

**Diagnosis**:
```bash
# Check if gateway is running
ps aux | grep hermes_cli

# Check cron ticker
tail -50 /root/.hermes/logs/agent.log | grep "Cron ticker"

# Verify schedule
cronjob list | grep <job_name>
```

## Debugging Workflow

### Step 1: Check Task Status
```bash
cronjob list
# Look for: last_status, last_delivery_error, state
```

### Step 2: Review Logs
```bash
# Recent logs
tail -200 /root/.hermes/logs/agent.log

# Specific job logs
tail -200 /root/.hermes/logs/agent.log | grep "cron_<job_id>"

# WeChat errors
tail -200 /root/.hermes/logs/agent.log | grep "Weixin"
```

### Step 3: Verify Configuration
```bash
# Check skills vs tools
ls /root/.hermes/skills/

# Verify job config
cronjob list | grep -A 20 "<job_id>"
```

### Step 4: Test Manually
```bash
# Run task manually
cronjob run --job_id <job_id>

# Monitor execution
tail -f /root/.hermes/logs/agent.log | grep "cron_<job_id>"
```

### Step 5: Check Gateway Stability
```bash
# Look for restarts
tail -500 /root/.hermes/logs/agent.log | grep -E "(restart|Gateway stopped)"

# Check memory
free -h

# Check disk space
df -h
```

## Common Fixes

### Fix 1: Optimize Content
- Reduce news count (30→15 international, 20→10 domestic)
- Split into 4 sub-tasks (not 1 long push)
- Each push ≤ 3000 chars
- Use bullet points, avoid long paragraphs

### Fix 2: Fix Skill Configuration
```yaml
# Wrong ❌
skills: ["web-search", "send-message", "macro-policy-impact-analysis"]

# Correct ✅
skills: ["macro-policy-impact-analysis", "multi-perspective-macro-analysis"]
```

### Fix 3: Reduce Execution Time
- Serial sub-tasks (not parallel)
- 15-30 second intervals between sub-tasks
- Total execution < 3 minutes

### Fix 4: Check Gateway
```bash
# Restart gateway if needed
# (Usually automatic, but check logs)
tail -100 /root/.hermes/logs/agent.log | grep "Gateway"
```

## Critical: Verify Real-Time Data Before Execution

**Root Cause Pattern**: Cron jobs outputting "模拟环境" (simulated environment) or fake data

**Diagnosis Steps**:
1. **Check skill configuration**: Verify skill names match available skills (e.g., `web-search` vs `web_search`, `send-message` vs `send_message`)
2. **Test search tools manually**: Run `web_search(query="test")` to confirm real-time data retrieval works
3. **Inspect API configurations**: Check `.env` for valid API keys (not `***` placeholders)
4. **Verify backend services**: Test local Firecrawl service (`curl http://localhost:8080/v1/search?query=test`)
5. **Check fallback status**: If primary service fails, ensure backup API keys are configured

**Common Issues**:
- **Skill name mismatch**: Using `web-search` instead of `web_search` (snake_case)
- **Invalid API keys**: `FIRECRAWL_SECONDARY_API_KEY=***` (placeholder, not real key)
- **Local service down**: Firecrawl local service not running or missing endpoints
- **Blocked responses**: Search API returning empty results due to anti-bot blocking

**Pre-Execution Checklist**:
- [ ] Skills exist in `skills_list` output
- [ ] API keys are valid (not `***`)
- [ ] Local services are running (if using local backend)
- [ ] Manual test search returns real data
- [ ] Fallback configuration is complete (primary + secondary URLs + keys)

**Post-Failure Verification**:
- Always verify output contains real URLs and timestamps
- Check for "模拟环境" or "模拟数据" warnings in output
- Cross-reference news items with external sources
- If output looks fake, immediately test search tools manually

## Quick Reference

| Symptom | Likely Cause | Quick Fix |
|---------|-------------|-----------|
| "Skill not found" | Tools listed as skills | Remove tools from skills list |
| WeChat timeout | Message too long | Split into 4 sub-tasks |
| Gateway restarts | Execution > 3 min | Reduce content, shorten intervals |
| Task not running | Gateway down | Check `ps aux \| grep hermes_cli` |
| No logs | Wrong job_id | Verify with `cronjob list` |