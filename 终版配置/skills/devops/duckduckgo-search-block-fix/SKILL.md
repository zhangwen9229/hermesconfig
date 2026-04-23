---
name: duckduckgo-search-block-fix
description: Fix DuckDuckGo anti-bot blocking by switching to paid search backend
version: 1.0.0
author: hermes-agent
license: MIT
metadata:
  hermes:
    tags: [search, duckduckgo, firecrawl, troubleshooting, devops]
    related_skills: [web-tools-configuration, api-key-management]
---

# Troubleshooting and Fixing DuckDuckGo Search Block

When web_search encounters "DuckDuckGo: Blocked by anti-bot measures" errors, switch to a paid search backend like Firecrawl.

## Detection

### Log Indicators
```
2026-04-19 13:24:17 warn [:]: DuckDuckGo: Encountered anti-bot measures, retrying... {"attempt":4,"term":"..."}
2026-04-19 13:24:18 error [:]: DuckDuckGo: Blocked by anti-bot measures {"term":"..."}
```

### Error Pattern
- Multiple retry attempts (attempt:4, attempt:5, etc.)
- Final error: `DuckDuckGo: Blocked by anti-bot measures`
- Stack trace shows `ddgSearch` function failing

## Root Cause
DuckDuckGo's free tier has strict anti-bot measures that block automated searches, especially from:
- Cloud/IP addresses
- High-frequency requests
- Specific geographic regions

## Solution: Switch to Firecrawl Backend

### Step 1: Configure config.yaml
Add the web backend section to the config file:

```yaml
web:
  backend: firecrawl
```

### Step 2: Configure .env with Dual Service Support
Add both local and official Firecrawl service configurations:

```bash
# Local Docker service (primary)
FIRECRAWL_PRIMARY_URL=http://localhost:8080/v1
FIRECRAWL_PRIMARY_API_KEY=your-local-api-key

# Official Firecrawl API (secondary - fallback)
FIRECRAWL_SECONDARY_URL=https://api.firecrawl.dev/v1
FIRECRAWL_SECONDARY_API_KEY=your-official-api-key
```

### Step 3: Verify Configuration
Check that both files are updated using standard config inspection commands.

### Step 3: Verify Configuration
Check that both files are updated using standard config inspection commands.

### Step 4: Test Search
Test with a Python script:

```python
from firecrawl import Firecrawl
client = Firecrawl()

result = client.search(query="test query")
print(f"Results: {len(result.model_dump()['web'])}")
```

## Alternative Backends

If Firecrawl doesn't work for your use case, consider:

| Backend | Best For | Setup |
|---------|----------|-------|
| **Tavily** | AI-focused search, fast results | `TAVILY_API_KEY` + `web.backend: tavily` |
| **Exa** | High-quality research, precise results | `EXA_API_KEY` + `web.backend: exa` |
| **Parallel** | Multi-engine aggregation, fault tolerance | `PARALLEL_API_KEY` + `web.backend: parallel` |

## Troubleshooting

### Issue: Firecrawl API returns Pydantic objects
**Solution**: Use `.model_dump()` to convert to dict:

```python
result = client.search(query="test")
data = result.model_dump()  # Convert Pydantic to dict
results = data['web']
```

### Issue: Still getting blocked
**Possible causes**:
- API key invalid or expired
- Rate limit exceeded (check billing plan)
- Network/firewall blocking API calls

**Solutions**:
1. Verify API key at https://firecrawl.dev/dashboard
2. Check Firecrawl status page
3. Try different backend (Tavily, Exa, Parallel)

### Issue: Config not taking effect
**Check**:
1. YAML syntax is valid (no indentation errors)
2. `.env` file is loaded (restart Hermes agent if needed)
3. Backend name matches exactly: `firecrawl`, `tavily`, `exa`, or `parallel`

## Prevention

To avoid future DuckDuckGo blocks:
- Always configure a paid backend when doing frequent searches
- Use Firecrawl for best anti-bot bypass capabilities
- Monitor search logs for early warning signs of rate limiting

## Advanced: Automatic Fallback to Official Service

The system automatically detects when the local Firecrawl service is blocked and falls back to the official service.

### Detection Keywords
The fallback mechanism checks for these keywords in responses:

**English:**
- `blocked by anti-bot`
- `duckduckgo: blocked`
- `captcha`
- `403 forbidden`
- `rate limit`
- `too many requests`
- `access denied`

**Chinese:**
- `反爬虫`
- `验证码`
- `访问被拒绝`
- `无法访问`
- `blocked`
- `blocked by`

### Implementation Details
The `_try_search_with_fallback()` function in `tools/web_tools.py`:
1. First attempts local service at `FIRECRAWL_PRIMARY_URL`
2. If response contains blocked keywords, retries with official service
3. Uses independent API keys for each service
4. Returns results from whichever service succeeds

### Troubleshooting Fallback
If fallback is not working:
1. Check logs for `[Firecrawl]` messages
2. Verify both API keys are valid
3. Confirm local service is running: `curl http://localhost:8080/v1/search`
4. Ensure Chinese keywords are included in detection logic