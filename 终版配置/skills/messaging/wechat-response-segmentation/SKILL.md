---
title: WeChat Response Segmentation
name: wechat-response-segmentation
category: messaging
description: Automatically segment WeChat responses exceeding 3000 characters to prevent truncation warnings
---

# WeChat Response Segmentation

## Overview
Automatically segment WeChat responses exceeding 3000 characters to prevent truncation warnings ("⚠️ Response truncated due to output length limit").

## Problem
WeChat has a hard limit of 4000 characters per message. When responses exceed this limit, users see truncation warnings. The solution is to proactively segment long responses into multiple messages.

## Configuration

### 1. Environment Variables
Set these in the environment configuration file:

```bash
# WeChat reply rules - Chinese (SINGLE-LINE FORMAT REQUIRED)
# ⚠️ IMPORTANT: .env files do NOT support multi-line values!
# Use | as delimiter for list items instead of numbered lists
WEIXIN_REPLY_RULES=请遵守以下回复规则：|1.如果回复内容超过 3000 个字符，请主动分段，每段不超过 3000 字符 |2.在段落之间使用空行分隔 |3.每个段落应该是完整的语义单元（如完整的章节、完整的分析点） |4.避免在句子中间或代码块中间断开 |5.如果内容很长，可以在开头说明"以下内容分为 X 段" |6.保持 Markdown 格式完整，不要在代码块中间断开

# Enable intelligent segmentation
WEIXIN_INTELLIGENT_SEGMENTATION=true

# Enable multiline split mode (recommended)
WEIXIN_SPLIT_MULTILINE_MESSAGES=true
```

### ⚠️ Critical .env Syntax Rules

**DO NOT use**:
- Numbered lists (1., 2., 3.) on separate lines
- Bullet points (-) on separate lines
- Multi-line string values

**DO use**:
- Single-line format with `|` as delimiter
- Escape quotes if needed: `\"`

**Example of common error**:
```bash
# ❌ WRONG - Causes "python-dotenv could not parse statement" errors
WEIXIN_REPLY_RULES=请遵守以下回复规则：
1. 如果回复内容超过 3000 个字符，请主动分段
2. 在段落之间使用空行分隔

# ✅ CORRECT - Single-line with pipe delimiter
WEIXIN_REPLY_RULES=请遵守以下回复规则：|1.如果回复内容超过 3000 个字符，请主动分段 |2.在段落之间使用空行分隔
```

### 2. Segmentation Scripts

**Python version** (recommended): `~/.hermes/scripts/weixin_response_segmenter.py`
- Smart segmentation by paragraphs
- Preserves code blocks intact
- Adds segment markers

**Bash version**: `~/.hermes/scripts/weixin_segmenter.sh`
- Simple paragraph-based splitting
- Faster for basic use cases

## Usage

### Automatic Segmentation (Recommended)
After restarting the gateway, all WeChat responses over 3000 characters will be automatically:
1. Detected for length
2. Split by semantic boundaries (paragraphs, code blocks)
3. Marked with segment indicators
4. Sent as separate messages

**Example output:**
```
【微信回复 - 共 2 段】
（每段不超过 3000 字符）

--- 第 1 段 / 共 2 段 (2978 字符) ---
第一段的内容...

--- 第 2 段 / 共 2 段 (188 字符) ---
第二段的内容...
```

### Manual Testing
```bash
# Test Python segmenter
python3 ~/.hermes/scripts/weixin_response_segmenter.py /path/to/response.txt

# Test Bash segmenter
bash ~/.hermes/scripts/weixin_segmenter.sh /path/to/response.txt /path/to/output.txt
```

### Prompt-Based Segmentation
Add to your system prompt:
```
请遵守 WeChat 回复规则：
- 如果回复超过 3000 字符，请主动分段
- 每段之间用空行分隔
- 保持语义完整性
- 避免在代码块中间断开
```

## Configuration Options

### Adjust Threshold
Edit `~/.hermes/scripts/weixin_response_segmenter.py`:
```python
MAX_CHARS = 3500  # Adjust to smaller value (WeChat limit: 4000)
```

### Enable/Disable
In the environment configuration:
```bash
WEIXIN_INTELLIGENT_SEGMENTATION=true   # Enable
WEIXIN_INTELLIGENT_SEGMENTATION=false  # Disable
```

### Split Mode
In the environment configuration:
```bash
WEIXIN_SPLIT_MULTILINE_MESSAGES=true   # Split mode - each segment as separate message (recommended)
WEIXIN_SPLIT_MULTILINE_MESSAGES=false  # Compact mode - try to merge into single message
```

## Verification

### 1. Check Environment Variables
Verify that the WeChat variables are set correctly in the configuration.

```bash
# Check WeChat reply rules configuration
grep "^WEIXIN_REPLY_RULES" /root/.hermes/.env

# Verify segmentation settings
grep -E "^WEIXIN_INTELLIGENT|^WEIXIN_SPLIT" /root/.hermes/.env
```

### 2. Test Segmenter Manually
```bash
# Create test file with long content
python3 -c "print('A' * 4000)" > /tmp/test_long.txt

# Run segmenter
python3 ~/.hermes/scripts/weixin_response_segmenter.py /tmp/test_long.txt
```

**Expected output**:
```
Original content: 4000 characters
Content exceeds 3000 characters, segmenting...
Segmented into 2 chunks
【微信回复 - 共 2 段】
（每段不超过 3000 字符）

--- 第 1 段 / 共 2 段 (2981 字符) ---
AAAAA...
...
```

### 3. Send Test Message to WeChat
Send to WeChat bot:
```
请测试一下分段功能，回复一段较长的内容（超过 3000 字符）。
```

**Verify**:
- ✅ Segment markers appear: `【微信回复 - 共 X 段】`
- ✅ Each segment under 3000 characters
- ✅ Semantic integrity maintained
- ✅ No truncation warnings
- ✅ Gateway restarts without parsing errors

### 4. Verify Gateway Logs
```bash
# Check for any python-dotenv parsing errors
hermes gateway logs | grep -i "dotenv\|parse"
```

**No errors = Configuration is correct**

## Troubleshooting

### Issue: "python-dotenv could not parse statement" errors on gateway restart

**Root Cause**: Environment variables with multi-line values or numbered lists (1., 2., 3.) cause parsing errors.

**Solution**:
1. Identify problematic lines from error message (e.g., "line 427")
2. Check if the variable contains multi-line content
3. Convert to single-line format using `|` as delimiter

**Example Fix**:
```bash
# Find the problematic line
sed -n '427p' /root/.hermes/.env

# Edit to use single-line format
# Replace numbered list items with pipe-delimited single line
```

**Verification**:
```bash
# Restart gateway - should show no parsing errors
hermes gateway restart
```

### Issue: Messages still show truncation after segmentation
**Solution**:
1. Restart gateway: `hermes gateway restart`
2. Check environment variables are set
3. View gateway logs for errors

### Issue: Segment markers don't appear
**Possible causes**:
- Response content under 3000 characters
- Segmenter not properly integrated
- Configuration not loaded

**Solution**:
1. Test segmenter manually
2. Verify script output is normal
3. Check gateway configuration
4. Confirm `WEIXIN_INTELLIGENT_SEGMENTATION=true`

### Issue: Code blocks are being broken
**Solution**:
1. Segmenter is optimized to preserve code blocks
2. Ensure proper Markdown format (``` for code blocks)
3. If issues persist, check if code blocks span multiple lines

## Performance

### Segmentation Latency
- Python version: ~0.1-0.5 seconds (depends on content length)
- Bash version: ~0.05-0.2 seconds

### Memory Usage
- Segmenter scripts: ~10-50MB (depends on content length)
- Impact on gateway performance: Negligible

### Recommended Configuration
- **Long content analysis**: Use split mode (`WEIXIN_SPLIT_MULTILINE_MESSAGES=true`)
- **Short conversations**: Use compact mode (`WEIXIN_SPLIT_MULTILINE_MESSAGES=false`)

## Best Practices

### 1. Prompt Optimization
Add to system prompt:
```
作为 WeChat 助手，请遵守以下回复规则：
- 如果回复超过 3000 字符，请主动分段
- 每段之间用空行分隔
- 保持语义完整性
- 避免在代码块中间断开
- 如果内容很长，在开头说明"以下内容分为 X 段"
```

### 2. Content Planning
When generating long content:
1. **Plan ahead**: Evaluate content length before starting response
2. **Proactive segmentation**: If expecting >3000 characters, segment proactively
3. **Add markers**: Indicate "以下内容分为 X 段" at the beginning

### 3. Testing Strategy
Regularly test segmentation:
```bash
# Create long content test
python3 -c "print('A' * 4000)" > /tmp/test_long.txt
python3 ~/.hermes/scripts/weixin_response_segmenter.py /tmp/test_long.txt
```

## Technical Details

### Segmentation Logic
1. **Paragraph-first splitting**: Uses double newlines (`\n\n`) as split points
2. **Code block preservation**: Doesn't break within code blocks
3. **Semantic integrity**: Avoids breaking in the middle of sentences
4. **Automatic marking**: Adds "第 X 段 / 共 Y 段" markers to each segment

### Markers Format
```
【微信回复 - 共 X 段】
（每段不超过 3000 字符）

--- 第 1 段 / 共 X 段 (N 字符) ---
Content...

--- 第 2 段 / 共 X 段 (N 字符) ---
Content...
```

## Files
- **Full documentation**: `~/.hermes/weixin_smart_segmentation.md`
- **Quick start guide**: `~/.hermes/weixin_quick_start.md`
- **Python script**: `~/.hermes/scripts/weixin_response_segmenter.py`
- **Bash script**: `~/.hermes/scripts/weixin_segmenter.sh`
- **Gateway code**: `~/.hermes/hermes-agent/gateway/platforms/weixin.py`

## Related Skills
- `wechat-message-truncation` - Troubleshooting WeChat message truncation
- `macro-policy-impact-analysis` - Framework for macroeconomic analysis (user preference)
- `ai-industry-news-analysis` - Daily AI industry news analysis framework
