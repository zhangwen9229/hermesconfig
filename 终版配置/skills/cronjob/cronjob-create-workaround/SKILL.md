---
name: cronjob-create-workaround
category: cronjob
description: 创建定时任务时遇到 TypeError 的标准解决方案 - 两步创建法
---

# 🚀 定时任务创建 Bug 标准解决方案

## ⚠️ 问题描述

当使用 `cronjob create` 命令时，如果出现以下错误：
```
TypeError: '<=' not supported between instances of 'str' and 'int'
```

**原因**：系统 `create()` 函数无法正确处理 `model` 和 `repeat` 参数。

## ✅ 标准解决方案：两步创建法

### 第 1 步：创建任务（**不传** `model` 和 `repeat`）

```bash
cronjob create \
  --name "任务名称" \
  --prompt "任务描述" \
  --schedule "0 13 * * *" \
  --skills skill1,skill2
```

**关键**：
- ❌ **不要传** `--repeat` 参数（系统默认 `forever`）
- ❌ **不要传** `--model` 参数
- ✅ 可以传 `--name`, `--prompt`, `--schedule`, `--skills`

### 第 2 步：更新任务（添加 `model` 等参数）

```bash
cronjob update \
  --job_id <从第 1 步输出中获取> \
  --repeat forever \
  --model '{"model":"model-name","provider":"provider"}'
```

**关键**：
- ✅ 使用 `update()` 添加 `model`、`repeat` 等参数
- ✅ `update()` 函数可以正确处理这些参数

## 📋 完整示例：创建每日 21 点分析任务

### 第 1 步：创建
```bash
cronjob create \
  --name "五家企业每日深度分析" \
  --prompt "每日 21:00 执行：为特斯拉、小鹏汽车、美团、比亚迪、英伟达抓取当日新闻，四视角分析..." \
  --schedule "0 13 * * *" \
  --skills macro-policy-impact-analysis,multi-perspective-macro-analysis
```

### 第 2 步：更新（可选）
```bash
cronjob update \
  --job_id 71fa63343ae5 \
  --repeat forever
```

## 🕐 时区转换规则

**重要**：Cron 使用 **UTC 时间**，北京时间 = UTC+8

| 北京时间 | UTC 时间 | Cron 表达式 |
|---------|---------|------------|
| 09:00   | 01:00   | `0 1 * * *` |
| 13:00   | 05:00   | `0 5 * * *` |
| 21:00   | 13:00   | `0 13 * * *` |
| 22:00   | 14:00   | `0 14 * * *` |

**快速记忆**：北京时间 - 8 = UTC 时间

## ✅ 验证步骤

创建后检查：
```bash
cronjob list
```

确认：
- ✅ `state: scheduled`（已激活）
- ✅ `enabled: true`（已启用）
- ✅ `next_run_at`：正确时间
- ✅ `repeat: forever`（如果更新了）

## 🎯 何时使用此方法

**必须使用两步法的情况**：
- ✅ 需要指定 `--model` 参数
- ✅ 需要指定 `--repeat` 参数（虽然默认是 forever）
- ✅ `create()` 报错 `TypeError`

**可以直接创建的情况**：
- ✅ 不需要 `model` 参数
- ✅ 接受默认 `repeat: forever`
- ✅ 简单任务

## 🚫 常见错误

### 错误 1：在 create 时传 repeat
```bash
# ❌ 错误
cronjob create --name "test" --schedule "0 13 * * *" --repeat forever
```

### 错误 2：在 create 时传 model
```bash
# ❌ 错误
cronjob create --name "test" --schedule "0 13 * * *" --model '{"model":"..."}'
```

### 错误 3：忘记时区转换
```bash
# ❌ 错误：以为 21 点是 0 21 * * *（实际是 UTC 时间）
# ✅ 正确：21 点北京时间 = 13 点 UTC → 0 13 * * *
```

## 📌 关键要点

1. **create 时只传基础参数**：`name`, `prompt`, `schedule`, `skills`
2. **update 时传高级参数**：`model`, `repeat`
3. **时区转换**：北京时间 - 8 = UTC
4. **先创建，后验证**：用 `cronjob list` 确认状态

## 🔗 相关资源

- GitHub Issue: #6709
- 相关技能：`cronjob-debugging`, `cronjob-tool-bug-workaround`