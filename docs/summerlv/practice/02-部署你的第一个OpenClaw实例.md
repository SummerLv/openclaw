# 02 - 部署你的第一个 OpenClaw 实例

> 目标：安装、配置并启动 OpenClaw Gateway，让 Agent 跑起来。

## 安装方式

### 方式一：npm 全局安装（推荐新手）

```bash
npm install -g openclaw@latest
```

### 方式二：从源码构建

```bash
cd openclaw
pnpm build
# 构建产物在 dist/ 目录
```

### 方式三：macOS 桌面应用

下载最新的 macOS App，自带 Gateway 管理。

## 首次启动

### 1. 初始化配置

```bash
# 交互式引导设置
openclaw setup
```

引导会帮你配置：
- 模型 Provider（OpenAI / Anthropic / 其他）
- API Key
- 消息频道（Telegram / WhatsApp / Discord 等）

### 2. 启动 Gateway

```bash
# 前台运行（开发调试用）
openclaw gateway run

# 后台运行
openclaw gateway run --bind loopback --port 18789 --force
```

### 3. 验证运行状态

```bash
# 检查 Gateway 状态
openclaw channels status --probe

# 检查端口监听
ss -ltnp | rg 18789

# 查看 Gateway 日志
tail -n 120 /tmp/openclaw-gateway.log
```

## 核心配置文件

配置文件位于 `~/.config/openclaw/openclaw.json`（或项目级 `.openclaw.json`）。

### 最小可用配置

```json5
{
  // 模型 Provider 配置
  models: {
    providers: {
      openai: {
        apiKey: "sk-..."  // 或通过环境变量 OPENAI_API_KEY
      }
    }
  },
  
  // 默认模型
  agents: {
    defaults: {
      model: "openai/gpt-4o"
    }
  },
  
  // Gateway 配置
  gateway: {
    auth: {
      mode: "shared-secret",  // 本地开发推荐
      token: "your-secret-token"
    }
  }
}
```

### 常用配置命令

```bash
# 设置配置项
openclaw config set models.providers.openai.apiKey sk-...
openclaw config set agents.defaults.model openai/gpt-4o
openclaw config set gateway.mode local

# 查看当前配置
openclaw config get

# 诊断配置问题
openclaw doctor

# 自动修复常见问题
openclaw doctor --fix
```

## Gateway 连接模式

| 模式 | 绑定地址 | 适用场景 |
|------|---------|---------|
| `loopback` | `127.0.0.1:18789` | 本地开发（默认） |
| `lan` | `0.0.0.0:18789` | 局域网访问 |
| `tailscale` | Tailscale IP | 远程安全访问 |

## 连接客户端

Gateway 启动后，可以用以下方式连接：

1. **CLI**：`openclaw chat` — 命令行聊天
2. **WebChat**：浏览器访问 `http://localhost:18789/__openclaw__/webchat/`
3. **macOS App**：自动发现本地 Gateway
4. **IM 频道**：配置 Telegram/Discord 等 Bot 连接

## 重启 Gateway

```bash
# 停止旧进程
pkill -9 -f openclaw-gateway || true

# 后台启动
nohup openclaw gateway run --bind loopback --port 18789 --force \
  > /tmp/openclaw-gateway.log 2>&1 &
```

## 常见问题

### Gateway 启动失败

```bash
# 检查端口占用
lsof -i :18789

# 查看详细日志
openclaw gateway run --verbose
```

### API Key 无效

```bash
# 验证配置
openclaw doctor

# 重新设置
openclaw config set models.providers.openai.apiKey sk-...
```

### 频道连接不上

```bash
# 检查频道状态
openclaw channels status --probe

# 重新运行频道设置
openclaw setup
```

## 下一步

Gateway 跑起来后，继续阅读 [03-连接IM频道实战](03-连接IM频道实战.md) 接入你常用的聊天工具。

---

*参考官方文档：`docs/concepts/architecture.md`、`docs/gateway/protocol.md`*
