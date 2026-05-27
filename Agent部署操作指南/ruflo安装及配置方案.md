# Ruflo 安装及配置方案

> 多 Agent 自动化协作平台 | 版本: v3.7 | 更新: 2026-05-23

---

## 目录

- [一、概述](#一概览)
- [二、系统要求](#二系统要求)
- [三、安装方式](#三安装方式)
- [四、初始化配置](#四初始化配置)
- [五、Web UI 部署（可选）](#五web-ui-部署可选)
- [六、核心概念与配置](#六核心概念与配置)
- [七、验证安装](#七验证安装)
- [八、常用命令速查](#八常用命令速查)
- [九、故障排查](#九故障排查)

---

## 一、概述

Ruflo 是一个为 Claude Code 打造的多 Agent 自动化编排平台。安装后，它自动接管任务路由、Agent 调度、团队协作和学习优化。

### 核心能力

| 能力 | 说明 |
|:----|:------|
| 🤖 **100+ 专用 Agent** | 编码、测试、审查、安全、架构等，各司其职 |
| 🐝 **自动 Swarm 协调** | Agent 自动组队、协商、分工（支持 mesh/分层/环形拓扑） |
| 🧠 **自学习路由** | Q-Learning + Thompson Sampling，越用越聪明 |
| 🧩 **33 个插件** | 按需安装记忆、安全、DevOps 等能力 |
| 🔌 **MCP 原生支持** | 210+ 内置 MCP 工具，也可接入自定义 MCP 服务器 |
| 🌐 **Web 图形界面** | 聊天式交互 + Goal Planner 可视化规划 |

---

## 二、系统要求

### 硬件要求

| 项目 | 最低 | 推荐 |
|:----|:----|:-----|
| CPU | 双核 | 4 核以上 |
| 内存 | 4 GB | 8 GB+ |
| 磁盘 | 1 GB 可用 | 5 GB+（含模型缓存） |
| 网络 | 可访问 GitHub/npm | 宽带连接 |

### 软件依赖

| 软件 | 版本要求 | 备注 |
|:----|:--------|:-----|
| **Node.js** | v20+ | 必需，运行 ruflo 核心 |
| **npm** | v9+ | 包管理工具 |
| **Git** | v2.0+ | 克隆仓库 |
| **Claude Code** | 最新版 | `npm install -g @anthropic-ai/claude-code` |
| **Docker**（可选） | 最新版 | 自部署 Web UI 时需要 |

### 支持的 Agent CLI（安装至少一个）

Ruflo 会自动检测 PATH 中已安装的 CLI：

- `claude`（Claude Code）
- `codex`（OpenAI Codex）
- `copilot`（GitHub Copilot CLI）
- `openclaw` / `opencode` / `hermes` / `gemini` / `pi` / `cursor-agent` / `kimi` / `kiro-cli`

---

## 三、安装方式

### 方式一：一键安装（推荐）

适用于 macOS / Linux / WSL：

```bash
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash
```

包含完整 MCP + 诊断的安装：

```bash
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --full
```

### 方式二：npx 安装（全平台通用）

适用于 Windows（PowerShell / cmd）：

```bash
npx ruflo@latest init wizard
```

交互式向导会自动配置。

### 方式三：npm 全局安装

```bash
npm install -g ruflo@latest
npx ruflo@latest init
```

---

## 四、初始化配置

### 步骤 1：安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

验证安装：

```bash
claude --version
```

### 步骤 2：初始化 Ruflo

```bash
npx ruflo@latest init wizard
```

交互式向导会执行以下操作：

1. 检测已安装的 Agent CLI
2. 生成 `.claude/`、`CLAUDE.md`、settings 等配置文件
3. 注册 MCP 服务器
4. 安装默认钩子（Hooks）系统

### 步骤 3：注册 MCP 服务器（可选但推荐）

```bash
claude mcp add ruflo -- npx ruflo@latest mcp start
```

### 步骤 4：验证配置

检查状态：

```bash
npx ruflo@latest hooks intelligence --status
```

查看已安装的 Agent 列表：

```bash
npx ruflo@latest agents list
```

---

## 五、Web UI 部署（可选）

Ruflo 提供两个 Web 图形界面，部署方式如下：

### 方式 A：直接使用托管版本（零安装）

无需部署，直接打开浏览器使用：

| 界面 | 地址 | 用途 |
|:----|:-----|:-----|
| 💬 聊天 UI | [https://flo.ruv.io](https://flo.ruv.io) | 多模型聊天 + MCP 工具调用 |
| 🎯 Goal Planner | [https://goal.ruv.io](https://goal.ruv.io) | 可视化目标规划 + Agent 仪表盘 |

### 方式 B：自部署 Web UI（Docker）

适用于内网或私有部署。

```bash
# 克隆仓库
git clone https://github.com/ruvnet/ruflo.git
cd ruflo/ruflo/src/ruvocal

# 配置环境变量
cp .env .env.local
```

编辑 `.env.local`，至少配置：

```env
OPENAI_BASE_URL=https://openrouter.ai/api/v1
OPENAI_API_KEY=sk-or-v1-你的密钥
TASK_MODEL=qwen/qwen3.6-max-preview
PUBLIC_APP_NAME=RuFlo
PUBLIC_APP_DESCRIPTION="多Agent自动化协作平台"
```

启动：

```bash
# Docker 部署（含 MongoDB）
docker build -t ruvocal --build-arg INCLUDE_DB=true .
docker run -p 3000:3000 \
  -e OPENAI_BASE_URL=https://openrouter.ai/api/v1 \
  -e OPENAI_API_KEY=sk-or-v1-你的密钥 \
  -v ruvocal-data:/data \
  ruvocal
```

访问 `http://localhost:3000`。

---

## 六、核心概念与配置

### 1. Swarm（集群）

Agent 自动组队的模式：

| 拓扑 | 适用场景 | 特点 |
|:----|:--------|:-----|
| `hierarchical`（分层） | 编码任务（默认） | 女王 Agent 协调，防漂移 |
| `mesh`（网状） | 研究/探索 | 点对点平等通信 |
| `ring`（环形） | 流水线场景 | 按序传递 |

### 2. 共识算法

多 Agent 对结果达成一致的方式：

| 算法 | 容错能力 | 说明 |
|:----|:--------|:-----|
| Raft | 1 个节点故障 | 标准主从共识 |
| Byzantine (BFT) | f < n/3 恶意节点 | 拜占庭容错 |
| Weighted | 女王 3x 权重 | 加权投票 |

### 3. 插件系统

核心插件一览：

| 插件 | 功能 |
|:----|:-----|
| `ruflo-core` | 基础 — 服务器、健康检查、插件发现 |
| `ruflo-swarm` | 多 Agent 团队协调 |
| `ruflo-autopilot` | 自动执行循环 |
| `ruflo-workflows` | 可复用多步任务模板 |
| `ruflo-rag-memory` | 智能检索增强记忆 |
| `ruflo-federation` | 跨机器 Agent 安全通信 |
| `ruflo-security-audit` | 漏洞扫描与修复 |
| `ruflo-testgen` | 自动生成测试用例 |
| `ruflo-jujutsu` | Git 差异分析、风险评分 |
| `ruflo-sparc` | SPARC 五阶段开发方法论 |

安装插件：

```bash
# Claude Code 插件方式
/plugin install ruflo-swarm@ruflo

# 或 CLI 方式
npx ruflo@latest plugins install @claude-flow/plugin-swarm
```

### 4. 智能路由配置

Ruflo 默认自动路由，也可手动指定模型复杂度：

```javascript
// 路由配置示例
swarm_init({
  topology: "hierarchical",  // 分层结构
  maxAgents: 8,              // 最大 Agent 数
  strategy: "specialized",   // 专业化分工
  consensus: "raft"          // 共识算法
})
```

### 5. MCP 工具管理

Web UI 中添加自定义 MCP 服务器：

1. 打开聊天界面
2. 点击输入框旁的 **MCP (n)** 按钮
3. 点击 **Add Server**
4. 输入 MCP 端点（HTTP/SSE/stdio）
5. 工具自动加入并行执行流程

---

## 七、验证安装

运行以下命令确认安装成功：

```bash
# 检查版本
npx ruflo@latest --version

# 查看 Agent 列表
npx ruflo@latest agents list

# 检查插件状态
npx ruflo@latest plugins list

# 测试 Swarm
npx ruflo@latest swarm test

# Web UI 访问（如已部署）
# 打开浏览器访问 http://localhost:3000
```

---

## 八、常用命令速查

### 安装与初始化

| 命令 | 说明 |
|:----|:-----|
| `npx ruflo@latest init wizard` | 交互式初始化向导 |
| `npx ruflo@latest init` | 快速初始化 |
| `claude mcp add ruflo -- npx ruflo@latest mcp start` | 注册 MCP 服务器 |

### Agent 管理

| 命令 | 说明 |
|:----|:-----|
| `npx ruflo@latest agents list` | 列出所有 Agent |
| `npx ruflo@latest swarm init` | 初始化 Swarm |
| `npx ruflo@latest swarm test` | 测试 Swarm 协调 |

### 插件管理

| 命令 | 说明 |
|:----|:-----|
| `npx ruflo@latest plugins list` | 列出已安装插件 |
| `npx ruflo@latest plugins install <name>` | 安装插件 |
| `/plugin install <name>@ruflo` | Claude Code 内安装插件 |

### 监控与调试

| 命令 | 说明 |
|:----|:-----|
| `npx ruflo@latest hooks intelligence --status` | 检查智能路由状态 |
| `npx ruflo@latest verify` | 校验安装完整性 |

---

## 九、故障排查

### 常见问题

| 问题 | 原因 | 解决方法 |
|:----|:----|:---------|
| `command not found: ruflo` | 未安装或 PATH 问题 | 重装：`npm install -g ruflo@latest` |
| MCP 服务器连接失败 | Claude Code 未安装 | `npm install -g @anthropic-ai/claude-code` |
| Web UI 无法启动 | MongoDB 未运行或端口占用 | 检查 Docker 状态，确认 3000 端口空闲 |
| Agent 未自动路由 | Hooks 未正确安装 | 重新运行 `npx ruflo@latest init` |
| 插件安装失败 | 网络问题或版本冲突 | 检查网络，更新到最新版 |

### 日志查看

```bash
# 查看运行日志
npx ruflo@latest --verbose

# 查看 MCP 服务器日志
claude mcp logs ruflo
```

---

> **参考链接**
> - 官方仓库: [https://github.com/ruvnet/ruflo](https://github.com/ruvnet/ruflo)
> - Web UI: [https://flo.ruv.io](https://flo.ruv.io)
> - Goal Planner: [https://goal.ruv.io](https://goal.ruv.io)
> - 文档: [https://github.com/ruvnet/ruflo/blob/main/docs/USERGUIDE.md](https://github.com/ruvnet/ruflo/blob/main/docs/USERGUIDE.md)
