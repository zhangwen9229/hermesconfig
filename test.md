version: '3.8'

services:
  # --- 核心服务：API ---
  firecrawl:
    image: ghcr.io/firecrawl/firecrawl:latest
    container_name: firecrawl-api
    restart: always
    environment:
      # 端口改回官方默认 8080（你之前写 3002 导致无法访问）
      - PORT=8080
      - HOST=0.0.0.0
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_RABBITMQ_URL=amqp://guest:guest@mq:5672
      - JWT_SECRET=xxxxxx
      - USE_DB_AUTHENTICATION=false
      # 新增：让 Firecrawl 连接内部 Playwright
      - PLAYWRIGHT_URL=http://playwright-service:3000
    ports:
      # 端口映射改为 8080:8080
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
      mq:
        condition: service_started
      # 增加依赖 playwright
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
      # 新增：Worker 也需要连接浏览器渲染
      - PLAYWRIGHT_URL=http://playwright-service:3000
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

  # --- 浏览器引擎：Playwright (修复版，可正常运行)
  playwright-service:
    image: mcr.microsoft.com/playwright:v1.58.2-noble
    container_name: firecrawl-playwright
    restart: always
    # 修复：启动正确的 Playwright Server（Firecrawl 必须用这个）
    command: npx playwright install chromium && npx playwright run-server --host=0.0.0.0 --port=3000
    ports:
      # 开放内部端口 3000
      - "3000:3000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  firecrawl_db_data:
  firecrawl_redis_data: