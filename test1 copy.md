version: '3.8'

services:
  # --- 核心服务：API ---
  firecrawl:
    image: ghcr.io/firecrawl/firecrawl:latest
    container_name: firecrawl-api
    restart: always
    environment:
      - PORT=8080
      - HOST=0.0.0.0
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_RABBITMQ_URL=amqp://guest:guest@mq:5672
      - JWT_SECRET=xxxxxx
      - USE_DB_AUTHENTICATION=false
      # ✅ 修正：Firecrawl 原生要求 ws:// CDP协议，不是 http://
      - PLAYWRIGHT_URL=ws://playwright-service:3000
      # 👇 👇 👇 【关键修复】自动初始化数据库表（解决你现在的报错）
      - AUTO_MIGRATE=true
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
      mq:
        condition: service_started
      playwright-service:
        condition: service_healthy

  # --- 工作节点 ---
  worker:
    image: ghcr.io/firecrawl/firecrawl:latest
    container_name: firecrawl-worker
    restart: always
    environment:
      - TYPE=worker
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_RABBITMQ_URL=amqp://guest:guest@mq:5672
      # ✅ 同步修正 ws 协议
      - PLAYWRIGHT_URL=ws://playwright-service:3000
    depends_on:
      - redis
      - db
      - mq
      - playwright-service

  # --- 数据库 ---
  db:
    image: postgres:16-alpine
    container_name: firecrawl-db
    restart: always
    environment:
      - POSTGRES_USER=firecrawl
      - POSTGRES_PASSWORD=firecrawl_pass
      - POSTGRES_DB=firecrawl
    volumes:
      - firecrawl_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U firecrawl"]
      interval: 5s
      timeout: 5s
      retries: 5

  # --- 缓存 ---
  redis:
    image: redis:7-alpine
    container_name: firecrawl-redis
    restart: always
    volumes:
      - firecrawl_redis_data:/data

  # --- MQ 服务 ---
  mq:
    image: rabbitmq:3-management-alpine
    container_name: firecrawl-mq
    restart: always

  # --- 浏览器引擎（你已拉取镜像，我保留完全可用版）
  playwright-service:
    image: mcr.microsoft.com/playwright:v1.58.2-noble
    container_name: firecrawl-playwright
    restart: always
    # ✅ 新增Docker权限：Chromium无头必须，解决崩溃、空回复
    cap_add:
      - SYS_ADMIN
    ipc: host
    init: true
    ports:
      - "3000:3000"
    # ✅ 修正启动命令：预安装+后台常驻服务，加-y免交互
    command: /bin/sh -c "npx -y playwright install chromium && npx playwright run-server --host=0.0.0.0 --port=3000"
    # ✅ 彻底废弃错误HTTP /health，改用TCP端口连通性健康检查（官方标准）
    healthcheck:
      test: ["CMD", "nc", "-zv", "localhost", "3000"]
      interval: 15s
      timeout: 5s
      retries: 5
      start_period: 60s # 给足chromium安装启动时间

volumes:
  firecrawl_db_data:
  firecrawl_redis_data: