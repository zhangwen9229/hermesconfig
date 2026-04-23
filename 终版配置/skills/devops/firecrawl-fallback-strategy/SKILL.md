---
name: firecrawl-fallback-strategy
category: devops
description: Firecrawl 主备双活降级策略 - 当本地服务被搜索引擎 block 时自动降级到官方服务
---

# Firecrawl 主备双活降级策略

## 场景描述
当本地 Firecrawl 服务（Docker 部署）的内部搜索引擎（如 DuckDuckGo）被 anti-bot 措施 block 时，自动降级到官方 Firecrawl 服务。

**关键问题**：
- 本地服务本身是健康的（HTTP 200 OK）
- 但内部搜索返回错误："DuckDuckGo: Blocked by anti-bot measures."
- 传统健康检查无法检测这种"逻辑 block"

## 配置步骤

### 1. 环境变量配置（必须）

**双重配置原则**：
```bash
# 第一部分：Hermes 要求的标准配置（用于启用 web tool）
FIRECRAWL_API_KEY=你的官方 API Key
FIRECRAWL_API_URL=https://api.firecrawl.dev/v1

# 第二部分：降级策略专用配置
FIRECRAWL_PRIMARY_URL=http://localhost:8080/v1
FIRECRAWL_PRIMARY_API_KEY=本地服务 Key（或任意值）
FIRECRAWL_SECONDARY_URL=https://api.firecrawl.dev/v1
FIRECRAWL_SECONDARY_API_KEY=你的官方 API Key
```

**关键点**：
- `FIRECRAWL_API_KEY` + `FIRECRAWL_API_URL`：**必须配置**，否则 Hermes 不会启用 web tool
- `FIRECRAWL_PRIMARY_*` + `FIRECRAWL_SECONDARY_*`：用于降级策略逻辑
- 缺少标准变量时，web tool 根本不会启动，降级策略无从谈起
- 本地和官方服务使用各自独立的 API Key
- 删除重复的 `FIRECRAWL_API_URL` 定义（会导致后者覆盖前者）

**常见陷阱**：只配置降级变量而不配置标准变量，导致 web tool 无法启用。

### 2. 实现降级逻辑

**核心思路**：
1. 定义被 block 关键词列表（blocked, anti-bot, captcha, 403, etc.）
2. 每次请求后检查响应内容是否包含这些关键词
3. 如果被 block，自动重试备用服务
4. 记录失败历史，连续失败后自动切换

**失败检测关键词**：
- blocked
- anti-bot
- captcha
- 403
- forbidden
- rate limit
- too many requests
- cloudflare
- access denied
- bypass
- duckduckgo

**降级策略**：
1. 优先尝试主服务（本地 Docker）
2. 如果响应包含 block 关键词，立即切换到备用服务（不等待连续失败）
3. 如果备用服务也被 block，抛出异常
4. 每次请求都先尝试主服务，仅当检测到 block 时才切换

**第一性原理分析**：
1. **核心目标**：确保搜索请求能成功返回结果，无论本地服务是否被 block
2. **关键事实**：
   - `Firecrawl` SDK 的 `.search()` 方法内部封装了 HTTP 请求，但**不暴露**响应内容供外部检查
   - `_try_search_with_fallback()` 使用 `requests.post()` 直接调用 API，**可以检查响应内容**
   - 本地服务返回 HTTP 200 但内容被 block，这是**业务层问题**，不是 HTTP 层问题
3. **方案对比**：
   | 方案 | 复杂度 | 侵入性 | 可维护性 |
   |------|--------|--------|----------|
   | 修改 `web_search_tool` 直接调用 `_try_search_with_fallback()` | 低 | 中 | 高 |
   | 修改 `Firecrawl` 客户端初始化注入降级逻辑 | 高 | 高 | 低 |
4. **推荐方案**：修改 `web_search_tool` 函数，理由：
   - 最小改动：只在工具函数中替换调用路径
   - 职责清晰：降级逻辑在工具层，不在客户端层
   - 易于测试：可以直接调用 `_try_search_with_fallback()` 验证
   - 符合单一职责：`Firecrawl` 客户端只负责封装 API，降级策略由工具层处理

**配置冲突修复**（重要教训）：
- **问题**：`.env` 中定义两次 `FIRECRAWL_API_URL` 会导致后者覆盖前者
  ```bash
  FIRECRAWL_API_URL=http://localhost:8080/v1  # 本地服务（被覆盖）
  FIRECRAWL_API_URL=https://api.firecrawl.dev/v1  # 官方服务（实际使用）
  ```
- **解决**：使用独立变量名
  ```bash
  FIRECRAWL_PRIMARY_URL=http://localhost:8080/v1
  FIRECRAWL_PRIMARY_API_KEY=你的本地服务 Key
  FIRECRAWL_SECONDARY_URL=https://api.firecrawl.dev/v1
  FIRECRAWL_SECONDARY_API_KEY=你的官方服务 Key
  ```
- **验证**：检查 `.env` 文件确认没有重复定义
- **关键**：必须同时配置标准变量（`FIRECRAWL_API_KEY` + `FIRECRAWL_API_URL`）以启用 web tool

**中文关键词检测**：添加中文反爬虫关键词（反爬虫、验证码、访问被拒绝、无法访问），因为官方服务可能返回中文错误信息

**关键修复**（2026-04-19）：Firecrawl 返回的 `data` 字段可能是 list 类型而非 dict，需要在 `_try_search_with_fallback` 中统一返回 dict 格式：

**JSON 序列化修复**（2026-04-20）：Firecrawl SDK 的 `.search()` 方法返回 `SearchData` 对象，不是 JSON 可序列化的，需要在 `web_search_tool` 中添加 `model_dump()` 转换：
```python
# 确保返回的是 dict 格式
if isinstance(response_json, dict):
    return response_json
elif isinstance(response_json, list):
    return {"data": {"web": response_json}}
```
否则 `_extract_web_search_results` 会因调用 `.get()` 方法而报错：`'list' object has no attribute 'get'`

**日志路径修复**（2026-04-19）：
- 问题：`cleanup_logs.py` 使用 `~/.hermes/hermes-agent/logs`，但实际日志在 `~/.hermes/logs`
- 解决：
  1. 创建符号链接：`ln -s ~/.hermes/logs ~/.hermes/hermes-agent/logs`
  2. 修正 `cleanup_logs.py` 中的 `LOG_DIR` 路径为 `~/.hermes/logs`
- 验证：`python3 /root/.hermes/scripts/cleanup_logs.py` 应输出 `清理完成：删除 0 个文件，释放 0KB`

**调试技巧**：检查 `~/.hermes/logs/agent.log` 确认 `web_search_tool` 函数被调用，降级逻辑生效

**常见陷阱**：如果搜索请求没有走降级逻辑，可能原因：
1. 没有重启 hermes gateway 以加载新代码
2. 直接调用 `web_search` 工具而不是通过 hermes 对话让模型调用
3. 模型决定不需要搜索（使用了已有知识回答）

**降级函数实现**：
```python
def _try_search_with_fallback(query: str, primary_url: str, secondary_url: str, 
                              primary_key: str, secondary_key: str) -> dict:
    services = [
        {'name': 'primary', 'url': primary_url, 'key': primary_key},
        {'name': 'secondary', 'url': secondary_url, 'key': secondary_key}
    ]
    
    for service in services:
        try:
            print(f"[Firecrawl] 使用 {service['name']} 服务：{service['url']}")
            
            response = requests.post(
                service['url'] + '/search',  # 添加 /search 后缀
                json={"query": query, "limit": 5},  # 使用正确的参数格式
                headers={
                    "Content-Type": "application/json",
                    "Authorization": f"Bearer {service['key']}"
                },
                timeout=30
            )
            
            if response.status_code != 200:
                print(f"[Firecrawl] {service['name']} 服务 HTTP 错误：{response.status_code}")
                continue
            
            response_json = response.json()
            if _is_blocked_response(str(response_json)):
                print(f"[Firecrawl] {service['name']} 服务被反爬虫拦截，尝试备用服务")
                continue
            
            return response_json
            
        except Exception as e:
            print(f"[Firecrawl] {service['name']} 服务请求失败：{e}")
            continue
    
    raise Exception(f"所有 Firecrawl 服务均不可用")
```

**验证方法**：
1. 检查 `_is_blocked_response` 函数可以正确处理 SearchData 对象
2. 确认 `FIRECRAWL_PRIMARY_URL` 和 `FIRECRAWL_SECONDARY_URL` 已设置
3. 通过 hermes 对话让模型调用搜索工具（不是直接调用 `web_search` 工具）
4. 检查日志确认 `web_search_tool` 函数被调用，降级逻辑生效

**JSON 序列化问题修复**（2026-04-20）：
- **问题**：`_get_firecrawl_client().search()` 返回 `SearchData` 对象，不是 JSON 可序列化的
- **错误**：`Object of type SearchData is not JSON serializable`
- **解决**：在 `web_search_tool` 中添加 `model_dump()` 转换
  ```python
  if hasattr(response, 'model_dump'):
      response = response.model_dump()
  elif hasattr(response, '__dict__'):
      response = response.__dict__
  ```
- **验证**：重启 hermes gateway 后，日志不再出现 JSON 序列化错误

### 3. 集成到现有代码

在 `web_tools.py` 中修改 `web_search_tool` 函数，添加降级包装器。

## 工作流程

```
1. 用户发起搜索请求
   ↓
2. 尝试本地服务 (FIRECRAWL_PRIMARY_URL + FIRECRAWL_PRIMARY_API_KEY)
   ↓
3. 本地服务返回响应
   ↓
4. 检查响应内容是否包含 "blocked" / "anti-bot" 等关键词
   ├─ 是 → 自动切换到官方服务 (FIRECRAWL_SECONDARY_URL + FIRECRAWL_SECONDARY_API_KEY)
   └─ 否 → 返回结果
   ↓
5. 如果官方服务也被 block，抛出异常
```

**测试验证**：
1. 启动本地服务：`docker run -d -p 8080:3000 firecrawl/app`
2. 验证本地服务：`curl http://localhost:8080/v1 -X POST -d '{"url": "https://duckduckgo.com/?q=test"}'`
3. 测试降级：`python -c "from tools.web_tools import web_search_tool; print(web_search_tool('test'))"`
4. 预期日志：`[Firecrawl] 使用 primary 服务：http://localhost:8080/v1`

## 关键特性

| 特性 | 说明 |
|------|------|
| **响应内容检测** | 检测 HTTP 200 但内容被 block 的情况 |
| **实时降级** | 每次请求都检测，发现 block 立即切换 |
| **简化逻辑** | 移除连续失败统计，避免误判 |
| **日志记录** | 方便调试和监控 |

## 调试技巧

1. **启用 DEBUG 日志**：
   ```bash
   export WEB_TOOLS_DEBUG=true
   ```

2. **手动测试降级**：
   - 模拟本地服务连续失败
   - 观察日志中的降级提示

3. **监控失败率**：
   - 定期检查失败历史记录
   - 调整阈值参数

## 常见问题

**Q: 为什么不用传统健康检查？**
A: 因为本地服务本身是健康的（HTTP 200），问题出在内部搜索引擎被 block，传统健康检查无法检测。

**Q: 冷却机制的作用是什么？**
A: 避免在本地服务不稳定时频繁切换，导致性能下降和 API 调用浪费。

**Q: 如何判断降级是否生效？**
A: 查看日志中的降级提示消息。

- **调试技巧**：检查 `~/.hermes/logs/agent.log` 确认 `web_search_tool` 函数被调用，降级逻辑生效
- **常见陷阱**：如果搜索请求没有走降级逻辑，可能原因：
  1. 没有重启 hermes gateway 以加载新代码
  2. 直接调用 `web_search` 工具而不是通过 hermes 对话让模型调用
  3. 模型决定不需要搜索（使用了已有知识回答）
**解决方法**：
1. 完全重启 hermes 服务
2. 在对话中发送需要实时信息的问题（如"搜索今天的天气"）
3. 检查 `~/.hermes/logs/agent.log` 中的日志

## 备份策略

修改前务必备份源文件：
```bash
cp /root/.hermes/hermes-agent/tools/web_tools.py \
   /root/.hermes/hermes-agent/tools/web_tools.py.bak.20260419_1459
```

**备份文件必须保留**，便于快速回滚。

**回滚命令**（如需）：
```bash
cp /root/.hermes/hermes-agent/tools/web_tools.py.bak.20260419_1459 \
   /root/.hermes/hermes-agent/tools/web_tools.py
systemctl restart hermes-gateway
```

## 相关文件

- `tools/web_tools.py` - 核心 Web 工具（**已修改**）
- `tools/browser_providers/firecrawl.py` - Firecrawl 浏览器提供者
- `.env` - 环境变量配置

**修改记录**（2026-04-20）：
1. 添加 `_is_blocked_response()` 函数，检测 block 错误
2. 添加 `_get_fallback_firecrawl_client()` 函数，获取备用服务客户端
3. 添加 `_try_search_with_fallback()` 函数，实现降级逻辑
4. 修改 `web_search_tool()` 函数，添加降级调用和 `model_dump()` 转换
5. 备份文件：`web_tools.py.bak.20260419_1459`

**关键修复**（2026-04-20）：
- **问题**：`_get_firecrawl_client().search()` 返回 `SearchData` 对象，不是 JSON 可序列化的
- **解决**：在 `web_search_tool` 中添加 `model_dump()` 转换
  ```python
  if hasattr(response, 'model_dump'):
      response = response.model_dump()
  elif hasattr(response, '__dict__'):
      response = response.__dict__
  ```

**关键修复**（2026-04-21）：\n- **问题**：本地 Firecrawl 服务内部调用 DuckDuckGo 时被 block，错误以**异常形式抛出**，不是正常响应\n- **发现**：之前的代码只检查响应内容，没有检查异常消息，导致降级逻辑未触发\n- **解决**：在 `except` 块中捕获异常，检查异常消息是否包含 blocked 关键词\n- **修改文件**：\n  - `_try_search_with_fallback()` 函数\n  - `_try_scrape_with_fallback()` 函数\n- **关键点**：\n  - 只检测 blocked 相关错误，其他错误重新抛出（避免误降级）\n  - 本地服务返回 HTTP 200 但内部搜索引擎被 block，这是业务层问题\n  - 必须同时检查响应内容和异常消息\n\n**关键修复**（2026-04-22）：\n- **问题**：降级逻辑没有触发，日志显示本地服务失败后没有切换到官方服务\n- **根因**：环境变量从 `.env` 文件加载失败\n  - `web_tools.py` 在模块导入时调用 `load_dotenv()`\n  - 但 Hermes Gateway 进程启动时，环境变量**没有从 .env 文件加载到进程环境**\n  - 导致 `os.getenv("FIRECRAWL_PRIMARY_URL")` 返回 `None`\n  - `_try_scrape_with_fallback()` 函数直接 `return None`，降级逻辑完全没执行\n- **证据**：\n  ```\n  2026-04-22 16:49:31 - 第一次 scrape 失败（本地服务）\n  2026-04-22 16:49:34 - 第二次 scrape 开始（还是本地服务，没有切换）\n  ```\n- **解决**：修改 `web_tools.py` 的环境变量加载逻辑，尝试多个可能的 `.env` 路径\n- **关键点**：\n  - 尝试多个可能的 `.env` 路径\n  - 使用 `override=True` 确保覆盖已有环境变量\n  - 添加日志确认加载成功\n  - **必须先重启 Hermes Gateway 才能生效**