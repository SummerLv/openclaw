# 06 - Agent 与记忆

> 官方文档：`docs/concepts/agent-loop.md`、`docs/concepts/memory.md`、`docs/concepts/context-engine.md`、`docs/concepts/multi-agent.md`

## Agent Loop（运行循环）

每条消息进来后，Agent 的完整处理流程：

```
intake（接收消息）
  ↓
context assembly（上下文组装）
  ↓
model inference（LLM 推理）
  ↓
tool execution（工具执行，循环直到完成）
  ↓
streaming reply（流式回复给用户）
  ↓
persistence（持久化会话状态）
```

**关键特性：每个 session 的 runs 是串行的**，防止工具/会话竞争，保持历史一致性。

### 五步详解

| 步骤 | 做什么 |
|------|-------|
| `agent` RPC | 验证参数，解析 session，立即返回 `{runId, acceptedAt}` |
| `agentCommand` | 解析模型，加载技能快照，调用 pi-agent-core |
| `runEmbeddedPiAgent` | 串行化执行，构建 pi session，订阅事件，强制超时 |
| `subscribeEmbeddedPiSession` | 把 pi 事件桥接为 OpenClaw agent 流（tool/assistant/lifecycle） |
| `agent.wait` | 等待 lifecycle end/error，返回最终状态 |

### 超时配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `agent.wait` 超时 | 30s | 仅等待超时，不中止 Agent |
| Agent 运行超时 | 48h | `agents.defaults.timeoutSeconds` |
| LLM 空闲超时 | 120s | 无响应时中止 LLM 请求 |

## 上下文引擎（Context Engine）

每次 Agent 运行前，上下文引擎组装完整的 "记忆 + 对话历史" 传给 LLM：

```
ingest（摄入原始对话）
  ↓
assemble（组装上下文，含记忆注入）
  ↓
compact（压缩：会话过长时触发）
  ↓
afterTurn（轮次结束后处理）
```

**压缩（Compaction）**：对话超过 token 限制时，上下文引擎会自动压缩历史，保留关键信息并触发记忆写入。

## Hook 拦截点

插件可以在 Agent Loop 的关键节点介入：

```
before_model_resolve   ← 覆盖 provider/model
before_prompt_build    ← 注入额外上下文
before_agent_reply     ← 返回合成回复或静默
  ↓ [LLM 推理]
before_tool_call       ← 拦截/修改工具参数
after_tool_call        ← 变换工具结果
tool_result_persist    ← 变换工具结果写入记录
  ↓ [回复生成]
agent_end              ← 检查最终消息列表
```

## 记忆系统

**核心设计**：记忆 = 普通 Markdown 文件，无隐藏状态，完全透明可编辑。

### 三种记忆文件

| 文件 | 内容 | 加载时机 |
|------|------|---------|
| `MEMORY.md` | 长期记忆（偏好、决策、事实） | 每次 DM 会话开始 |
| `memory/YYYY-MM-DD.md` | 每日笔记 | 今天和昨天自动加载 |
| `DREAMS.md` | Dreaming 整合摘要（实验性） | 人工审阅用 |

文件默认在 `~/.openclaw/workspace/`

### 记忆工具

Agent 内置两个记忆工具：
- `memory_search`：语义搜索（配置 embedding provider 后支持向量+关键词混合搜索）
- `memory_get`：读取特定文件或行范围

### 三种记忆后端

| 后端 | 特点 |
|------|------|
| **Builtin（默认）** | SQLite，零依赖，开箱即用，支持混合搜索 |
| **QMD** | 本地优先 sidecar，支持重排序、查询扩展、跨目录索引 |
| **Honcho** | AI 原生，跨会话用户建模，多 Agent 感知 |

### Dreaming（实验性后台整合）

三阶段自动整合短期记忆为长期记忆：

| 阶段 | 频率 | 成本 | 目的 |
|------|------|------|------|
| **Light** | 每 6 小时 | 低 | 轻度整理短期信号 |
| **Deep** | 每日凌晨 3 点 | 高 | 晋升高质量内容到 MEMORY.md |
| **REM** | 每周 | 模式化 | 发现跨时间的规律模式 |

默认**关闭**，通过配置开启：

```bash
openclaw memory status
openclaw memory search "TypeScript 偏好"
openclaw memory index --force
```

### 压缩前自动记忆写入

会话压缩前，OpenClaw 自动提醒 Agent 把重要内容保存到记忆文件，**无需配置，默认开启**。

## 多 Agent

一个 Gateway 进程可以运行多个隔离 Agent：

### 单 Agent 模式（默认）

```
agentId = "main"
workspace = ~/.openclaw/workspace
sessions  = ~/.openclaw/agents/main/sessions
```

### 多 Agent 模式

每个 Agent 拥有独立的：
- Workspace（文件、AGENTS.md、SOUL.md、USER.md）
- agentDir（认证配置、模型注册）
- Session 存储

```bash
# 添加新 Agent
openclaw agents add work

# 验证 Agent 状态
openclaw agents status
```

**配置 bindings 路由入站消息到指定 Agent：**

```json
{
  "agents": {
    "list": [
      {
        "id": "work",
        "bindings": [
          { "channel": "slack" },
          { "channel": "msteams" }
        ]
      }
    ]
  }
}
```

**注意**：认证配置严格按 Agent 隔离，禁止复用 `agentDir`！

## 系统提示（System Prompt）结构

Agent 收到的系统提示由以下部分组成：

| 部分 | 来源 |
|------|------|
| Tooling | 可用工具描述 |
| Safety | 安全规则 |
| Skills | 加载的技能 |
| Workspace | Bootstrap 文件（AGENTS.md、SOUL.md、TOOLS.md、IDENTITY.md、USER.md、HEARTBEAT.md、MEMORY.md） |
| Current Date & Time | 当前时间 |
| Reply Tags | 回复格式规则 |
| Heartbeats | 心跳任务 |

## 下一步

→ [07-安全与部署](07-安全与部署.md)：了解 OpenClaw 的安全模型和部署方式
