# 帮助文档 / Help

## 本地启动 / Local Development

```shell
# 1. 克隆项目（如果需要）
git clone https://github.com/songquanpeng/one-api.git
cd one-api

# 2. 构建前端
cd web/default
npm install
npm run build
cd ../..

# 3. 构建后端
go mod download
go build -ldflags "-s -w" -o one-api

# 4. 运行
chmod u+x one-api
./one-api --port 3000 --log-dir ./logs
```

访问 http://localhost:3000/ 并登录：
- 用户名：`root`
- 密码：`123456`

**注意**：首次登录后务必修改默认密码！

## 环境变量 / Environment Variables

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `PORT` | 服务端口 | `3000` |
| `SQL_DSN` | 数据库连接字符串 | SQLite |
| `REDIS_CONN_STRING` | Redis 连接字符串 | - |
| `SESSION_SECRET` | 会话加密密钥 | - |
| `THEME` | UI 主题：`default`、`berry`、`air` | `default` |
| `NODE_TYPE` | `master` 或 `slave` 多机部署 | `master` |
| `RELAY_PROXY` | LLM 请求出口代理 | - |
| `CHANNEL_TEST_FREQUENCY` | 渠道健康检测间隔（秒） | - |

## Docker 部署 / Docker Deployment

```shell
# 构建镜像
docker build -t one-api:latest .

# 运行（SQLite）
docker run --name one-api -d --restart always -p 3000:3000 \
  -e TZ=Asia/Shanghai \
  -v /data/one-api:/data \
  justsong/one-api

# Docker Compose（MySQL + Redis）
docker-compose up -d
```

## 更多文档 / More

- [README](./README.md) - 项目完整文档
- [API 扩展](./API.md) - 如何添加自定义 API 扩展
