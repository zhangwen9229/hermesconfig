---
name: cron-job-management
description: Guide for managing scheduled tasks using crontab and cronjob tool
---

# Cron Job Management Skill

## Overview
This skill provides guidelines for managing scheduled tasks using `crontab` and the Hermes `cronjob` tool, including troubleshooting and best practices.

## When to Use

### Use `crontab` (Terminal) When:
- Creating simple, one-off scheduled tasks
- The `cronjob` tool fails with type errors (known bug)
- You need precise control over task scheduling
- Managing system-level tasks (log cleanup, data collection)

### Use `cronjob` Tool When:
- Creating complex, multi-step tasks requiring skills
- Tasks need to be managed through the Hermes interface
- You want automatic delivery to chat threads

## Common Patterns

### 1. Basic Crontab Entry
```bash
# Format: minute hour day-of-month month day-of-week command
0 2 * * * /path/to/script.sh >> /path/to/log.log 2>&1
# Example: Run cleanup_logs.py daily at 02:00
0 2 * * * /usr/bin/python3 /root/.hermes/scripts/cleanup_logs.py >> /root/.hermes/cron/output/cleanup_logs_$(date +%Y%m%d).log 2>&1
```

### 2. Adding/Removing Tasks
```bash
# View current tasks
crontab -l

# Add new task (preserves existing)
(crontab -l 2>/dev/null; echo "0 21 * * * /path/to/script.sh >> /path/to/log.log 2>&1") | crontab -

# Remove specific task
(crontab -l 2>/dev/null | grep -v "pattern_to_remove") | crontab -
# ⚠️ Warning: grep -v removes ALL lines matching pattern
```

### 3. Task Execution Testing
```bash
# Test script manually
/bin/bash /path/to/script.sh

# Check logs
cat /path/to/log.log
```

## Known Issues & Workarounds

### Issue: `cronjob` Tool Type Error
**Symptom**: Error `'<' not supported between instances of 'str' and 'int'`
**Cause**: Bug in `cronjob` tool parameter handling
**Workaround**: Use `crontab` via terminal instead

### Issue: Bash Scripts Cannot Call Hermes Tools
**Symptom**: `send_message: command not found` in bash script
**Cause**: Hermes tools (e.g., `send_message`, `web_search`) are not available in bash environment
**Solutions**:
1. **Data Collection Only**: Use bash for web scraping, save to files, then manually review
2. **Hybrid Approach**: Bash script collects data → generate report file → trigger manual send
3. **Alternative Tools**: Use system tools (curl, mail) for notifications instead of `send_message`

## Best Practices

### 1. Logging
Always redirect output to log files:
```bash
command >> /path/to/log.log 2>&1
```

### 2. Error Handling
Include error checking in scripts:
```bash
#!/bin/bash
set -e  # Exit on error
# Your commands here
```

### 3. Path Management
Use absolute paths for all commands and files:
```bash
# ✅ Good
/usr/bin/python3 /root/.hermes/scripts/script.py

# ❌ Bad
python3 scripts/script.py
```

### 4. Cron Syntax Reference
| Field | Allowed Values | Special Characters |
|-------|----------------|-------------------|
| Minute | 0-59 | `*`, `,`, `-`, `/` |
| Hour | 0-23 | `*`, `,`, `-`, `/` |
| Day of Month | 1-31 | `*`, `,`, `-`, `/` |
| Month | 1-12 | `*`, `,`, `-`, `/` |
| Day of Week | 0-7 (0=Sunday) | `*`, `,`, `-`, `/` |

**Examples**:
- `0 2 * * *` - Daily at 02:00
- `0 21 * * *` - Daily at 21:00
- `0 12 * * 3` - Every Wednesday at 12:00
- `*/15 * * * *` - Every 15 minutes

## Troubleshooting Checklist

- [ ] Verify script is executable: `chmod +x /path/to/script.sh`
- [ ] Check crontab syntax: `crontab -l`
- [ ] Test script manually: `/path/to/script.sh`
- [ ] Review logs: `cat /path/to/log.log`
- [ ] Ensure absolute paths are used
- [ ] Check file permissions
- [ ] Verify environment variables (cron has minimal env)

## Common Use Cases

### Daily Log Cleanup
```bash
0 2 * * * /usr/bin/python3 /root/.hermes/scripts/cleanup_logs.py >> /root/.hermes/cron/output/cleanup_logs_$(date +%Y%m%d).log 2>&1
```

### Daily News Collection
```bash
0 21 * * * /bin/bash /root/.hermes/scripts/daily_hot_news.sh >> /root/.hermes/cron/output/daily_news_$(date +%Y%m%d).log 2>&1
```

### Weekly Report Generation
```bash
0 12 * * 3 /bin/bash /root/.hermes/scripts/market_analysis.sh >> /root/.hermes/cron/output/market_outlook_$(date +%Y%m%d).log 2>&1
```

## Notes
- Cron jobs run in a minimal environment (no GUI, limited PATH)
- Use `>>` to append logs, `>` to overwrite
- `2>&1` redirects stderr to stdout
- Date formatting in filenames: `$(date +\%Y\%m\%d)`
- Escape special characters in crontab entries
