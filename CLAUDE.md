# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

One API is a unified API gateway for LLM providers, routing requests through the standard OpenAI API format to 40+ providers (OpenAI, Claude, Gemini, Azure, AWS Bedrock, ByteDance Doubao, Baidu Wenxin, Alibaba Qwen, DeepSeek, etc.).

**Stack**: Go (Gin) backend + React frontend + SQLite/MySQL + Redis, containerized with Docker.

## Build Commands

```shell
# Build Docker image (uses npm mirror + goproxy.cn for China)
docker build -t one-api:latest .

# Run with Docker Compose (MySQL + Redis + One API)
docker-compose up -d

# Run container directly (SQLite, simplest)
docker run --name one-api -d --restart always -p 3000:3000 \
  -e TZ=Asia/Shanghai \
  -v /data/one-api:/data \
  justsong/one-api
```

### Build Frontend (Local Development)

```shell
cd web/default && npm install && npm run build
# Themes: default, berry, air - each built independently
```

### Run from Binary

```shell
go mod download
go build -ldflags "-s -w" -o one-api
./one-api --port 3000 --log-dir ./logs
```

## Architecture

```
User → One API (relay) → LLM Provider
```

The relay system is the core: `controller/relay.go` handles incoming requests, then routes to the appropriate adaptor in `relay/adaptor/` based on the channel configuration.

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `relay/adaptor/` | 40+ LLM provider implementations (openai, anthropic, gemini, aws, doubao, baidu, ali, deepseek, etc.) |
| `controller/` | HTTP handlers for all API endpoints |
| `model/` | GORM database models (user, token, channel, ability, log, etc.) |
| `router/` | Gin router definitions |
| `middleware/` | Request ID, logging, CORS, session middleware |
| `common/` | Config, logger, Redis client, rate limiting |
| `monitor/` | Channel health monitoring and metrics |
| `web/` | React frontend with 3 themes (default, berry, air), built assets embedded at `web/build/` |

### Request Flow

1. Request hits Gin router (`router/`)
2. Middleware processes (auth, rate limit, logging)
3. Controller handles it (`controller/`)
4. For LLM requests: `relay/` module selects the appropriate adaptor
5. Adaptor converts OpenAI format to provider-specific format and calls the LLM

### Database

Uses GORM with SQLite (default) or MySQL/PostgreSQL. Models are in `model/`. Migrations are SQL scripts in `bin/migration_*.sql`.

## Key Files

- `main.go` - Entry point, initializes DB, Redis, i18n, Gin server
- `VERSION` - Version string embedded into binary at build time
- `Dockerfile` - Multi-stage: builds 3 React themes + Go binary, produces ~79MB Alpine image
- `docker-compose.yml` - MySQL 8.2 + Redis + One API
- `.env.example` - All environment variables documented

## Environment Variables

- `SQL_DSN` - Database connection (e.g., `oneapi:123456@tcp(db:3306)/one-api`)
- `REDIS_CONN_STRING` - Redis URL (e.g., `redis://redis`)
- `SESSION_SECRET` - Session encryption key
- `THEME` - UI theme: `default`, `berry`, or `air`
- `NODE_TYPE` - `master` or `slave` for multi-server deployment
- `RELAY_PROXY` - Outbound proxy for LLM requests
- `CHANNEL_TEST_FREQUENCY` - Auto-test channel health interval (seconds)

## Default Credentials

- Username: `root`, Password: `123456` — **change on first login**

## API Extension

Providers can be extended via `docs/API.md` which documents how to add custom API extensions.
