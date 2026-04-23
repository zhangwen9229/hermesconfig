---
name: cronjob-simulation-data-fix
category: cronjob
description: Fix cron jobs that output simulated data instead of real information
---

# Cron Job Simulation Data Fix

## Problem
Cron jobs output "模拟环境" (simulated environment) instead of real data.

## Root Cause
Environment variables from `.env` files are not automatically loaded in cron job processes.

## Solution

### Quick Fix
Add environment loading to tool modules:

```python
from dotenv import load_dotenv
import os

load_dotenv()
```

### Verification Steps

1. **Check search works manually**:
   - Run `web_search` with test query
   - Should return real results

2. **Verify environment variables**:
   - Check if FIRECRAWL variables are set
   - Ensure API keys are not placeholders

3. **Test cron job**:
   - Run job manually with `cronjob action=run`
   - Verify it returns real data

## Common Mistakes

- Using wrong skill names (e.g., `web-search` instead of `ai-industry-news-analysis`)
- Placeholder API keys (`***` instead of real keys)
- Local service not running or wrong endpoint

## Prevention

- Always test cron jobs manually before relying on them
- Verify API keys are valid (not `***`)
- Check skill names with `skills_list` command