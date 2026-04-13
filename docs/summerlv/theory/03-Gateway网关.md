# 03 - Gateway 网关

> 官方文档：`docs/gateway/protocol.md`、`docs/gateway/index.md`、`docs/concepts/architecture.md`

## Gateway 是什么

Gateway 是 OpenClaw 的**核心守护进程**，扮演以下角色：

- 所有消息面（WhatsApp、Telegram、Discord 等）的唯一拥有者
- 控制平面：管理所有客户端（CLI、Web UI、macOS 应用）的连接
- 数据平面：处理消息路由、Agent 调度、事件推送

**每台主机只运行一个 Gateway**，默认监听 `127.0.0.1:18789`。

## 启动与运维

```bash
# 启动 Gateway（默认端口 18789）
openclaw gateway --port 18789

# 强制关闭占用端口的进程再启动
openclaw gateway --force

# 验证健康状态
openclaw gateway status
openclaw logs --follow

# 验证频道连接状态
openclaw channels status --probe
```

**健康基线**：`Runtime: running` + `RPC probe: ok`

## 运行时模型

Gateway 是单进程、单端口复用的服务：

| 服务 | 路径/端点 |
|------|---------|
| WebSocket 控制/RPC | `ws://127.0.0.1:18789` |
| OpenAI 兼容 HTTP API | `/v1/models`、`/v1/chat/completions`、`/v1/responses` |
| 工具调用 HTTP | `/tools/invoke` |
| Canvas 宿主 | `/__openclaw__/canvas/` |
| A2UI 宿主 | `/__openclaw__/a2ui/` |
| Control UI | 同端口 |

## WS 线协议

### 消息格式

三种消息类型：

```json
// 请求（Client → Gateway）
{ "type": "req", "id": "xxx", "method": "agent", "params": {...} }

// 响应（Gateway → Client）
{ "type": "res", "id": "xxx", "ok": true, "payload": {...} }

// 事件推送（Gateway → Client）
{ "type": "event", "event": "agent", "payload": {...}, "seq": 1 }
```

### 握手流程

```
Gateway → Client:  connect.challenge (nonce + timestamp)
Client  → Gateway: connect 请求 (protocol 版本、role、auth token)
Gateway → Client:  hello-ok (protocol: 3, tickIntervalMs: 15000)
```

**connect 请求示例：**

```json
{
  "type": "req",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": { "id": "cli", "version": "1.2.3", "platform": "macos" },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "auth": { "token": "your-token" }
  }
}
```

### 连接生命周期

```mermaid
Client      →  Gateway: connect
Gateway     →  Client:  hello-ok
Gateway     →  Client:  event:presence
Gateway     →  Client:  event:tick (每 15s)

Client      →  Gateway: req:agent
Gateway     →  Client:  res:agent (runId, status: "accepted")
Gateway     →  Client:  event:agent (流式)
Gateway     →  Client:  res:agent (final, status, summary)
```

## 角色与范围

连接时客户端声明 `role`：

| Role | 用途 |
|------|------|
| `operator` | CLI、Web UI、macOS App，完整控制权 |
| `node` | iOS、Android、headless 设备节点 |

**Nodes 额外声明的能力（caps/commands）举例：**
- `canvas.*` — 可编辑 HTML 画布
- `camera.*` — 摄像头访问
- `screen.record` — 屏幕录制
- `location.get` — 位置获取

## 认证模式

| 模式 | 说明 |
|------|------|
| 共享密钥（默认） | `connect.params.auth.token` |
| Tailscale Serve | `gateway.auth.allowTailscale: true`，从请求头读取身份 |
| Trusted Proxy | `gateway.auth.mode: "trusted-proxy"`，非回环绑定 |
| None | `gateway.auth.mode: "none"`，仅限私有网络 |

## 配置热重载

默认模式 `hybrid`：
- 监听活跃配置文件路径
- 成功重载后**原子替换**内存快照
- 无需重启 Gateway

```bash
# 触发配置重载
openclaw gateway reload
```

## 事件类型

Gateway 会主动推送的事件：

| 事件 | 触发时机 |
|------|---------|
| `presence` | 频道连接状态变化 |
| `tick` | 每 15 秒心跳 |
| `agent` | Agent 运行流式输出 |
| `chat` | 新消息 |
| `health` | 健康状态变化 |
| `heartbeat` | Heartbeat 触发 |
| `cron` | 定时任务触发 |
| `shutdown` | Gateway 即将关闭 |

## 配置文件路径

```
~/.openclaw/openclaw.json    # 主配置文件
```

或通过 `OPENCLAW_CONFIG_PATH` 环境变量指定。

## 下一步

→ [04-插件系统](04-插件系统.md)：了解如何扩展 Gateway 的能力
