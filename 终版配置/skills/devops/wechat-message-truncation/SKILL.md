---
name: wechat-message-truncation
description: Troubleshooting and fixing WeChat bot message truncation issues
version: 1.0.0
author: Hermes Agent
tags: [wechat, weixin, messaging, truncation, troubleshooting, gateway]
---

# WeChat Bot Message Truncation Troubleshooting

This skill provides a systematic approach to diagnosing and fixing "Response truncated" warnings in WeChat (Weixin) bots. Use when your WeChat bot displays truncation warnings or messages are cut off unexpectedly.

## 🔍 Root Cause Analysis

### Primary Cause
- **WeChat iLink Bot API message limit: 4000 characters** (Unicode code points)
- Responses exceeding this limit are automatically split, but the split logic may not always prevent truncation warnings
- Default configuration uses "compact mode" which tries to merge messages, potentially triggering warnings

### Secondary Causes
- Gateway not restarted after config changes
- Incorrect environment variable name
- Model generating responses longer than platform limits
- Encoding issues causing character count discrepancies

## 🛠️ Diagnosis Steps

### Step 1: Check Current Configuration

Use the hermes CLI to inspect platform settings:
```bash
hermes config show --platform weixin
```

### Step 2: Verify Gateway Status

Check if WeChat gateway is running and review logs for truncation warnings.

### Step 3: Confirm Message Limit

The WeChat adapter defines `MAX_MESSAGE_LENGTH = 4000` in the gateway adapter. Messages exceeding this are split using the platform's built-in splitter.

## 🎯 Solutions

### Solution 1: Enable Multi-Line Split Mode (Recommended) ⭐

**When to use**: For long-form content, analysis reports, or any response likely to exceed 4000 characters.

**Steps**:
1. Add configuration to environment file
2. Restart the WeChat gateway
3. Test with a long response

**Effect**: 
- Each paragraph/blank line becomes a separate message bubble
- Prevents truncation warnings by staying under 4000 character limit per message
- Messages appear as multiple consecutive bubbles in WeChat

**Trade-off**: Long responses show as multiple bubbles instead of one continuous message.

### Solution 2: Fix .env Configuration Parsing Errors ⭐

**When to use**: When gateway restart fails with "python-dotenv could not parse statement" errors.

**Common Cause**: Environment variables with multi-line values cause parsing errors.

**Steps**:
1. Identify problematic lines (check error message for line numbers)
2. Convert multi-line values to single-line format
3. Use `|` as delimiter for list items instead of numbered lists (1., 2., 3.)

**Example Fix**:
```bash
# ❌ WRONG - Multi-line with numbered list (causes parsing errors)
WEIXIN_REPLY_RULES=请遵守以下回复规则：
1. 如果回复内容超过 3000 个字符，请主动分段
2. 在段落之间使用空行分隔

# ✅ CORRECT - Single-line with pipe delimiter
WEIXIN_REPLY_RULES=请遵守以下回复规则：|1.如果回复内容超过 3000 个字符，请主动分段 |2.在段落之间使用空行分隔
```

**Verification**:
```bash
# Check environment variables
grep "^WEIXIN_" /root/.hermes/.env

# Restart gateway (should show no parsing errors)
hermes gateway restart
```

**Pitfall**: `.env` files do NOT support multi-line string values. Always use single-line format with delimiters.

### Solution 2: Limit Output Length

**When to use**: When you prefer single-message format and can accept shorter responses.

**Steps**:
- Configure model to generate shorter responses
- Set token limits in model configuration

**Effect**: Model generates shorter responses that fit within WeChat limits.

### Solution 3: Prompt Engineering

**When to use**: For specific queries where you control the prompt.

**Add to prompt**:
```
Please keep responses concise, under 3000 Chinese characters.
If content is long, output in paragraphs with blank lines between sections.
```

**Effect**: Reduces response length at the source.

## 📋 Configuration Reference

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `WEIXIN_SPLIT_MULTILINE_MESSAGES` | `false` | Enable multi-line splitting (sends each paragraph as separate message) |
| `WEIXIN_ACCOUNT_ID` | — | iLink Bot account ID (required) |
| `WEIXIN_TOKEN` | — | iLink Bot token (required) |
| `WEIXIN_BASE_URL` | `https://ilinkai.weixin.qq.com` | API base URL |

### Config.yaml Platform Settings

```yaml
platforms:
  weixin:
    extra:
      split_multiline_messages: true  # Alternative to env var
```

## ✅ Testing the Fix

### Verification Steps

1. Confirm configuration is loaded
2. Restart gateway
3. Send test message asking for a long response to verify splitting works

### Expected Behavior

**Before fix**: Single message with truncation warning if >4000 chars

**After fix**: Multiple message bubbles, each <4000 chars, no warnings

## 🚨 Advanced Troubleshooting

### If Splitting Still Causes Issues

1. **Check for encoding problems**: Some characters may count differently
2. **Verify gateway is using new config**: Restart is required after config changes
3. **Check for other platform limits**: Some platforms have different constraints

### Platform-Specific Notes

- **WeChat (iLink Bot)**: 4000 characters, supports Markdown
- **Telegram**: 4096 characters (different split logic)
- **Discord**: 2000 characters per message (requires more aggressive splitting)

## 📁 Related Files

- Gateway adapter: `weixin.py` in gateway platforms directory
- Split function: `_split_text_for_weixin_delivery()` 
- Base splitter: `base.py` (split_message method)

## 📝 Quick Reference Card

```
Problem: "Response truncated" warning on WeChat bot

Fix:
  1. Add WEIXIN_SPLIT_MULTILINE_MESSAGES=true to environment
  2. Restart hermes gateway
  3. Test with long response

Verify:
  Check gateway logs for Weixin configuration
```

## 🎯 When to Use Each Solution

| Scenario | Recommended Solution |
|----------|---------------------|
| Long analysis reports | Solution 1 (multi-line split) |
| Quick Q&A responses | Solution 3 (prompt engineering) |
| Single-message preference | Solution 2 (limit output) |
| Production bot | Solution 1 + Solution 3 |
| Testing/debugging | Solution 1 |

## ⚠️ Common Pitfalls

1. **Forgetting to restart gateway**: Config changes require gateway restart
2. **Using wrong env var name**: Must be `WEIXIN_SPLIT_MULTILINE_MESSAGES` (not `WECHAT`)
3. **Assuming all platforms use 4000 limit**: Verify each platform's actual limit
4. **Not testing after config change**: Always send a long test message

## 🔗 Related Skills

- `hermes-agent` - General Hermes Agent usage
- `cron-job-debugging` - Systematic troubleshooting approach
