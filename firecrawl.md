version: '3.8'

services:
  # --- 核心服务：API ---
  firecrawl:
    image: ghcr.io/firecrawl/firecrawl:latest
    container_name: firecrawl-api
    restart: always
    environment:
      - PORT=3002
      - HOST=0.0.0.0
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_RABBITMQ_URL=amqp://guest:guest@mq:5672
      - JWT_SECRET=xxxxxx
      - USE_DB_AUTHENTICATION=false
    ports:
      - "3002:3002"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
      mq:
        condition: service_started

  # --- 工作节点 ---
  worker:
    image: ghcr.io/firecrawl/firecrawl:latest
    container_name: firecrawl-worker
    restart: always
    environment:
      - TYPE=worker
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      # 必须在 worker 这里也加上这行，否则 worker 启动时会去检查 docker
      - NUQ_DATABASE_URL=postgresql://firecrawl:firecrawl_pass@db:5432/firecrawl
      - NUQ_RABBITMQ_URL=amqp://guest:guest@mq:5672
    depends_on:
      - redis
      - db
      - mq


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

  # --- 修正：补全缺失的 MQ 服务，解决 undefined service "mq" 报错 ---
  mq:
    image: rabbitmq:3-management-alpine
    container_name: firecrawl-mq
    restart: always

  # --- 浏览器引擎：Playwright (如果需要渲染 JS) ---
  playwright-service:
    image: mcr.microsoft.com/playwright:v1.58.2-noble
    container_name: firecrawl-playwright
    restart: always
    # 必须添加 command，否则容器启动后会立即退出导致不断重启
    command: /bin/sh -c "npx playwright install-deps && sleep infinity"
    # 如果 Firecrawl Worker 需要远程连接它，通常需要开启以下端口（视具体调用逻辑而定）
    # ports:
    #   - "3000:3000" 
    # 增加健康检查，确保浏览器依赖安装完成
    healthcheck:
      test: ["CMD", "npx", "playwright", "--version"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  firecrawl_db_data:
  firecrawl_redis_data:
