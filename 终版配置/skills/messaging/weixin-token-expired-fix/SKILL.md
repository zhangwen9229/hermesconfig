---
name: weixin-token-expired-fix
category: messaging
description: Weixin (WeChat) token 失效导致发送消息失败的诊断和修复方案
---

# Weixin Token 失效修复方案

## 问题症状

**错误日志**：
```
iLink sendmessage error: ret=-2 errcode=None errmsg=unknown error
```

**特征**：
- 可以正常接收消息（inbound 正常）
- 发送消息失败（send failed）
- 错误码 `ret=-2`，无具体错误信息
- 重试多次后仍然失败

## 根本原因

Weixin iLink Bot API 的 token 已过期或失效，导致发送消息时返回通用错误 `unknown error`。

**常见触发场景**：
1. Token 长期未使用导致过期
2. 微信后台刷新了 token
3. 账户被重新配对
4. 服务器重启后 token 缓存失效

## 诊断步骤

### Step 1: 检查错误日志
```bash
tail -100 /root/.hermes/logs/agent.log | grep -E "(Weixin|send failed|ret=-2)"
```

### Step 2: 验证 token 文件存在
```bash
ls -la /root/.hermes/weixin/accounts/ae8e3a7b90dd@im.bot.json
```

### Step 3: 检查 token 内容
```bash
cat /root/.hermes/weixin/accounts/ae8e3a7b90dd@im.bot.json
```

### Step 4: 验证上下文 token
```bash
cat /root/.hermes/weixin/accounts/ae8e3a7b90dd@im.bot.context-tokens.json
```

## 解决方案

### 方案 1: 重新配对 Weixin（推荐）

**步骤**：

1. **删除旧账户数据**：
```bash
rm -f /root/.hermes/weixin/accounts/ae8e3a7b90dd@im.bot*.json
rm -f /root/.hermes/pairing/weixin-approved.json
```

2. **重启 Hermes Gateway**：
```bash
# 找到 hermes 进程
ps aux | grep "hermes.*chat" | grep -v grep

# 杀掉进程（替换 <pid>）
kill -9 <pid>

# 重新启动
hermes chat
```

3. **扫描二维码重新配对**：
- 启动后会显示配对二维码
- 使用微信扫描并输入配对码
- 确认授权

4. **验证配对成功**：
```bash
# 检查新 token 文件
ls -la /root/.hermes/weixin/accounts/

# 查看日志
tail -50 /root/.hermes/logs/agent.log | grep -i weixin
```

### 方案 2: 检查并更新 .env 配置

**检查配置**：
```bash
grep "^WEIXIN_" /root/.hermes/.env
```

**必需配置**：
```bash
WEIXIN_ACCOUNT_ID=ae8e3a7b90dd@im.bot
WEIXIN_TOKEN=你的新 token
WEIXIN_BASE_URL=https://ilinkai.weixin.qq.com
WEIXIN_CDN_BASE_URL=https://novac2c.cdn.weixin.qq.com/c2c
WEIXIN_DM_POLICY=pairing
WEIXIN_HOME_CHANNEL=o9cq80_odBcFVmip8-GGD_8MXnQs@im.wechat
```

**注意**：
- `.env` 中的 token 不会自动同步到账户文件
- 需要重新配对才能获取新的 token
- 修改配置后必须重启 gateway

## 验证修复

### 测试发送消息
```bash
# 在微信中发送一条消息
# 检查 hermes 是否能正常回复
```

### 检查日志
```bash
tail -50 /root/.hermes/logs/agent.log | grep -E "(Weixin|send.*success|inbound)"
```

**成功标志**：
- 看到 `inbound message` 日志
- 看到 `Sending response` 日志
- 没有 `send failed` 错误

## 预防措施

### 1. 定期监控日志
```bash
# 设置日志监控
watch -n 60 'tail -20 /root/.hermes/logs/agent.log | grep -E "(Weixin|send failed)"'
```

### 2. 配置告警
在 `.env` 中添加：
```bash
# WeChat message splitting configuration
WEIXIN_SPLIT_MULTILINE_MESSAGES=false
WEIXIN_INTELLIGENT_SEGMENTATION=true
```

### 3. 备份账户数据
```bash
# 定期备份
cp /root/.hermes/weixin/accounts/*.json ~/weixin_backup/
```

## 常见问题

**Q: 重新配对后 token 还是失效？**
A: 检查微信后台是否限制了设备数量，可能需要解绑其他设备。

**Q: 配对二维码不显示？**
A: 检查 `WEIXIN_DM_POLICY` 是否为 `pairing`，确认 gateway 已正确启动。

**Q: 日志中仍有 send failed 错误？**
A: 检查网络连接，确认能访问 `https://ilinkai.weixin.qq.com`。

**Q: 如何知道 token 什么时候会过期？**
A: Weixin token 过期时间不固定，建议每月检查一次日志，发现错误立即重新配对。

## 相关技能

- `wechat-message-truncation` - WeChat 消息截断问题
- `cronjob-debugging` - 定时任务调试

## 参考文档

- Weixin 官方文档：https://ilinkai.weixin.qq.com/
- Hermes Weixin 适配器：`/root/.hermes/hermes-agent/gateway/platforms/weixin.py`