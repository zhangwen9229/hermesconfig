---
name: cron-job-debugging
description: Systematic approach to diagnosing and fixing scheduled cron job failures in Hermes Agent
version: 1.0.0
author: Hermes Agent
tags: [cron, debugging, troubleshooting, automation, scheduled-tasks]
---

# Cron Job Debugging & Optimization

This skill provides a systematic framework for diagnosing and fixing scheduled cron job failures in Hermes Agent. Use when cron jobs execute but produce empty responses, errors, or unexpected output.

## 🔍 Root Cause Analysis Framework

### Step 1: Check Configuration
```bash
# List all jobs
hermes cron list

# Check specific job details
cat ~/.hermes/cron/jobs.json | jq '.jobs[] | select(.id == "JOB_ID")'
```

**Key fields to verify:**
- `schedule`: Cron expression (e.g., `0 0 * * *` = UTC 00:00 = Beijing 08:00)
- `last_status`: Should be `success`, not `error`
- `last_error`: Error message if failed
- `model`/`provider`: May be `null` if using defaults
- `enabled`: Should be `true`

### Step 2: Check Execution Output
```bash
# Find output directory
ls -la ~/.hermes/cron/output/

# Check job-specific output
ls -la ~/.hermes/cron/output/JOB_ID/

# Read the output file
cat ~/.hermes/cron/output/JOB_ID/YYYY-MM-DD_HH-MM-SS.md
```

**Common failure patterns:**
- `(No response generated)` → Model executed but produced empty output
- File exists but size = 0 → Task completed but no content
- File missing → Task didn't run at all

### Step 3: Identify Root Cause

**Pattern A: Empty Response**
```
last_error: "Agent completed but produced empty response (model error, timeout, or misconfiguration)"
```
**Causes:**
1. Date placeholders not replaced (e.g., `YYYY-MM-DD` stayed literal)
2. Search queries returned no results
3. Model response truncated or cut off
4. Prompt too complex for single execution

**Pattern B: Search Failure**
- Check if search queries used correct date format
- Verify API keys for search providers
- Check if time range filters too restrictive

**Pattern C: Model Configuration**
- `model: null` → Using default model (may be insufficient for complex tasks)
- `provider: null` → Using default provider
- Consider specifying explicit model for complex multi-step tasks

---

## 🛠️ Optimization Strategies

### Strategy 1: Fix Date Replacement Logic

**Problem:** Prompts use `YYYY年 MM月 DD日` placeholders that may not replace correctly

**Solution:** Hardcode the date in the prompt
```markdown
# Before (problematic)
"AI news YYYY年 MM月 DD日 latest"

# After (fixed)
"AI news 2026 年 04 月 19 日 latest"
```

**Implementation:**
1. Read current prompt
2. Replace all date placeholders with execution date
3. Update jobs.json with new prompt

### Strategy 2: Simplify Prompt Structure

**Problem:** Complex multi-step prompts cause model confusion

**Solution:** Break into numbered steps with clear inputs/outputs

```markdown
## Step 1: Data Collection
- [ ] Execute search queries
- [ ] Filter by time range
- [ ] Validate sources

## Step 2: Analysis
- [ ] Calculate metrics
- [ ] Apply filters
- [ ] Sort by relevance

## Step 3: Output Generation
- [ ] Follow template
- [ ] Include all sections
- [ ] Verify completeness
```

**Benefits:**
- Clearer execution path
- Easier debugging
- Better model compliance

### Strategy 3: Add Error Handling & Degradation

**Problem:** No fallback when data unavailable

**Solution:** Define explicit degradation paths

```markdown
## Error Handling

### Search Returns 0 Results
1. Expand time range to 48 hours
2. If still 0, use 7-day popular news
3. If still nothing, output "[SILENT]"

### Search Returns <5 Results
1. Lower validation requirement (≥1 source)
2. Mark as "unverified"
3. Continue with full report

### Model Response Truncated
1. Generate in steps (news first, then analysis)
2. Prioritize core conclusions
3. Mark as "partial output"
```

### Strategy 4: Add Execution Checklist

**Problem:** Model skips steps without verification

**Solution:** Include pre-output checklist

```markdown
## Execution Checklist
Before generating final report, confirm:
- [ ] Date replaced correctly
- [ ] Executed all search queries
- [ ] Filtered by time range
- [ ] Validated sources (≥2 per news)
- [ ] Sorted by relevance
- [ ] Covered all required topics
- [ ] Included all output sections
```

---

## 📋 Implementation Template

### Update jobs.json

```python
import json
import os
import re

# Read current config
jobs_path = os.path.expanduser('~/.hermes/cron/jobs.json')
with open(jobs_path, 'r', encoding='utf-8') as f:
    data = json.load(f)

# Update prompt
job = data['jobs'][0]
job['prompt'] = optimized_prompt

# Write back
with open(jobs_path, 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
```

### Strategy: Prompt Section Removal/Reordering

When removing sections from cron job prompts (e.g., removing investment recommendations):

```python\nimport re\n\n# Remove specific section with its content\nprompt = re.sub(\n    r'### \\d+\\.\\d+ Section Title\\s*```.*?```',\n    '',\n    prompt,\n    flags=re.DOTALL\n)\n\n# Renumber subsequent sections\nprompt = re.sub(r'### (\\d+)\\.(\\d+)', lambda m: f'### {int(m.group(1))}.{int(m.group(2)) - 1}', prompt)\n\n# Update checklist counts\nprompt = prompt.replace('已包含 6 个输出部分', '已包含 4 个输出部分')\n```\n\n**Use cases:**\n- Removing sensitive sections (investment advice, portfolio recommendations)\n- Simplifying prompts for model performance\n- Adapting output format for different audiences\n\n### Verify Changes\n\n```python\n# Read updated config\nwith open(jobs_path, 'r', encoding='utf-8') as f:\n    data = json.load(f)\n\n# Check key sections exist\nprompt = data['jobs'][0]['prompt']\nsections = ["第一步", "第二步", "第三步", "错误处理", "执行检查清单"]\nfor section in sections:\n    assert section in prompt, f\"Missing section: {section}\"\n\nprint(\"✅ All sections present\")\n```\n\n### Strategy: Cron Job Creation with System-Level Scheduling

When `cronjob` tool fails with type validation errors (e.g., `'<= not supported between instances of 'str' and 'int'`):

**Fallback approach:**
1. Create shell script in `~/.hermes/scripts/` for task execution
2. Add cron entry via `crontab -e` or write to `/etc/cron.d/`
3. Use `crontab -l` to verify installation

**Example cron entry (Beijing time 22:00 = 0 22 * * *):**
```bash\n0 22 * * * /root/.hermes/scripts/daily_macro_query.sh >> /var/log/macro_query.log 2>&1\n```

**Verification:**
```bash\ncrontab -l  # Should show your entry\n```

**Use cases:**
- When cronjob tool has type validation bugs
- When system-level scheduling is preferred
- When logging to file is required
- When multiple scripts need coordinated scheduling
import re

# Remove specific section with its content
prompt = re.sub(
    r'### \d+\.\d+ Section Title\s*```.*?```',
    '',
    prompt,
    flags=re.DOTALL
)

# Renumber subsequent sections
prompt = re.sub(r'### (\d+)\.(\d+)', lambda m: f'### {int(m.group(1))}.{int(m.group(2)) - 1}', prompt)

# Update checklist counts
prompt = prompt.replace('已包含 6 个输出部分', '已包含 4 个输出部分')
```

**Use cases:**
- Removing sensitive sections (investment advice, portfolio recommendations)
- Simplifying prompts for model performance
- Adapting output format for different audiences

### Verify Changes

```python
# Read updated config
with open(jobs_path, 'r', encoding='utf-8') as f:
    data = json.load(f)

# Check key sections exist
prompt = data['jobs'][0]['prompt']
sections = ["第一步", "第二步", "第三步", "错误处理", "执行检查清单"]
for section in sections:
    assert section in prompt, f"Missing section: {section}"

print("✅ All sections present")
```

---

## 🎯 Common Issues & Fixes

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| Empty response | Date placeholders not replaced | Hardcode date in prompt |
| No search results | Time range too restrictive | Expand to 48h or 7 days |
| Model timeout | Prompt too complex | Simplify structure, add steps |
| Partial output | Response truncated | Add checklist, prioritize sections |
| API errors | Missing credentials | Check `.env` for API keys |
| Wrong time | UTC vs local confusion | Verify cron expression timezone |

---

## 🚀 Testing

After making changes:

1. **Manual test:**
   ```bash
   hermes cron run JOB_ID
   ```

2. **Wait for next scheduled run:**
   - Check `next_run_at` in jobs.json
   - Verify execution at scheduled time

3. **Monitor output:**
   ```bash
   tail -f ~/.hermes/cron/output/JOB_ID/$(date +%Y-%m-%d)_*.md
   ```

---

## 📝 Best Practices

1. **Always include error handling** - Define what happens when data is unavailable
2. **Use explicit dates** - Don't rely on dynamic replacement
3. **Add execution checklists** - Prevent skipped steps
4. **Test manually first** - Verify before waiting for scheduled run
5. **Monitor first run** - Check output immediately after execution
6. **Keep prompts modular** - Separate data collection, analysis, output
7. **Document degradation paths** - Know what to do when things fail

---

## 🔗 Related Skills

- `hermes-agent` - General Hermes Agent usage
- `cronjob` - Cron job management tool
- `debugging` - Systematic debugging approach