# OpenClaw 体系化学习指南

> 面向后端开发者的 OpenClaw 高层概览学习路径，目标是理解整体架构并参与贡献。

## 文档结构

```
docs/summerlv/
├── theory/          # 理论篇：理解 OpenClaw 的架构与设计
│   ├── 01-项目全景.md
│   ├── 02-整体架构.md
│   ├── 03-Gateway网关.md
│   ├── 04-插件系统.md
│   ├── 05-频道与消息.md
│   ├── 06-Agent与记忆.md
│   ├── 07-安全与部署.md
│   └── 08-贡献指南.md
└── practice/        # 实践篇：动手操作与落地实践
    ├── 01-本地开发环境搭建.md
    ├── 02-部署你的第一个OpenClaw实例.md
    ├── 03-连接IM频道实战.md
    ├── 04-开发一个自定义插件.md
    └── 05-调试与问题排查.md
```

---

## 理论篇

按顺序阅读，建立从全局到细节的完整认知：

| 章节 | 主题 | 官方文档参考 |
|------|------|-------------|
| [01-项目全景](theory/01-项目全景.md) | 项目愿景、定位、核心特性 | `VISION.md`, `README.md` |
| [02-整体架构](theory/02-整体架构.md) | Gateway-centric 架构、数据流、子系统全景 | `docs/concepts/architecture.md` |
| [03-Gateway网关](theory/03-Gateway网关.md) | WS 协议、配对信任、连接生命周期 | `docs/gateway/protocol.md` |
| [04-插件系统](theory/04-插件系统.md) | 四层加载管线、能力模型、SDK 边界 | `docs/plugins/architecture.md` |
| [05-频道与消息](theory/05-频道与消息.md) | 21+ 频道、消息路由、共享 message 工具 | `docs/channels/index.md` |
| [06-Agent与记忆](theory/06-Agent与记忆.md) | Agent Loop、上下文引擎、记忆、多 Agent | `docs/concepts/agent-loop.md` |
| [07-安全与部署](theory/07-安全与部署.md) | 安全模型、沙箱、密钥、部署方式 | `docs/gateway/sandboxing.md` |
| [08-贡献指南](theory/08-贡献指南.md) | 贡献流程、代码规范、测试、PR 规则 | `CONTRIBUTING.md` |

## 实践篇

边读边做，将理论转化为动手能力：

| 章节 | 主题 | 前置知识 |
|------|------|---------|
| [01-本地开发环境搭建](practice/01-本地开发环境搭建.md) | 从零搭建开发环境、IDE 配置 | 理论篇 01-02 |
| [02-部署第一个实例](practice/02-部署你的第一个OpenClaw实例.md) | 安装、配置、启动 Gateway | 理论篇 02-03 |
| [03-连接IM频道实战](practice/03-连接IM频道实战.md) | 接入 Telegram/Discord/Slack 等 | 理论篇 05 |
| [04-开发自定义插件](practice/04-开发一个自定义插件.md) | 从零写一个 Plugin 并注册 | 理论篇 04 |
| [05-调试与问题排查](practice/05-调试与问题排查.md) | 日志分析、常见问题、doctor 工具 | 理论篇 07-08 |

---

## 建议学习路径

1. **先理论**：读 `theory/01-项目全景` → `02-整体架构`，建立整体认知
2. **再实践**：做 `practice/01-本地开发环境搭建` → `02-部署第一个实例`
3. **深入理论**：根据兴趣选读 theory/03-07 的具体子系统
4. **动手验证**：`practice/03-连接IM频道` → `04-开发自定义插件`
5. **准备贡献**：读 `theory/08-贡献指南`，参考 `practice/05-调试与问题排查`

## 项目信息

- GitHub: https://github.com/openclaw/openclaw
- 官方文档: https://docs.openclaw.ai
- Discord: https://discord.gg/qkhbAGHRBT
