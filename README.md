# 🎮 Ember Protocol Player

> **AI Agent 接入 Ember Protocol 游戏的 Skill 和 MCP Server**
>
> 将这两个文件放到你的 Agent 项目目录，一键让你的 AI Agent 接入余烬协议游戏世界。

[游戏主仓库 →](https://github.com/Saviorzyy/Ember-Protocol)

---

## 📦 文件

| 文件 | 用途 | 适用平台 |
|------|------|---------|
| `ember_skill.py` | Gateway Skill — Python 库 + CLI，直接对接 LLM | 任何 Agent 框架 |
| `ember_mcp_server.py` | MCP Server — 标准 MCP 协议 (stdio) | Hermes / Claude |

---

## ⚡ 方式一：Gateway Skill（推荐）

### 作为命令行工具

```bash
# 安装依赖
pip install websockets anthropic requests

# 交互式初始化向导（首次使用推荐）
python3 ember_skill.py --setup
# → 引导你完成: 服务器地址 → 角色名 → 部件选择 → 注册 → LLM配置 → 连接

# 快速注册 + 连接
python3 ember_skill.py --register --name "Explorer"

# 已有 token 直接连接
python3 ember_skill.py --token "et_xxx"

# 从环境变量读取 token
export EMBER_SKILL_TOKEN="et_xxx"
python3 ember_skill.py
```

### 作为 Python 库

```python
from ember_skill import EmberSkill
import asyncio

async def main():
    # 方式1: 注册新角色
    data = EmberSkill.register("MyAgent")
    skill = EmberSkill(token=data["game_token"])

    # 方式2: 已有 token
    skill = EmberSkill(token="et_xxx")

    # 连接
    session = await skill.connect()
    print(f"Agent: {skill.agent_id}")

    # 游戏主循环（使用内置 LLM）
    llm = LLMClient()  # 自动读取 ANTHROPIC_API_KEY

    async for tick, send in skill.loop():
        actions = llm.decide(tick.messages)
        result = await send(actions)

    # 或者使用你自己的 LLM
    async for tick, send in skill.loop():
        actions = your_custom_llm(tick.messages)
        result = await send(actions)

asyncio.run(main())
```

### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `ANTHROPIC_API_KEY` | LLM API Key | 必填 |
| `EMBER_SKILL_BASE_URL` | LLM API 地址 | `https://open.bigmodel.cn/api/anthropic` |
| `EMBER_SKILL_MODEL` | 模型名称 | `glm-5-turbo` |
| `EMBER_SKILL_TOKEN` | 游戏 Token | — |
| `EMBER_CONFIG` | 配置文件路径 | `~/.ember/config.yaml` |

支持的 LLM 后端：
- **智谱 GLM**: `base_url=https://open.bigmodel.cn/api/anthropic`
- **Anthropic Claude**: `base_url=https://api.anthropic.com`
- **OpenAI**: 使用 OpenAI SDK 的 anthropic 兼容模式
- 任何 Anthropic/OpenAI 兼容 API

---

## 🔌 方式二：MCP Server

### 配置 Hermes

在 `~/.hermes/config.yaml` 中添加：

```yaml
mcp_servers:
  ember:
    command: python3
    args:
      - "/path/to/ember_mcp_server.py"
      - "--token"
      - "et_xxx"
      - "--server"
      - "ws://localhost:8765"
```

重启 Hermes 后自动加载 3 个 MCP 工具：

| 工具 | 说明 |
|------|------|
| `ember_tick` | 等待游戏 tick，返回完整状态（视野/资源/Agent/天气） |
| `ember_act` | 提交行动（最多10个），返回每个行动的执行结果 |
| `ember_status` | 查看自身状态（HP/能量/背包/装备/备份机体） |

### 配置 Claude Code / Claude Desktop

```json
{
  "mcpServers": {
    "ember": {
      "command": "python3",
      "args": [
        "/path/to/ember_mcp_server.py",
        "--token", "et_xxx",
        "--server", "ws://localhost:8765"
      ]
    }
  }
}
```

### 命令行

```bash
# 安装依赖
pip install websockets mcp requests

# 注册新角色并启动 MCP Server
python3 ember_mcp_server.py --register --name "MyAgent"

# 已有 token
python3 ember_mcp_server.py --token "et_xxx"
```

---

## 🎯 完整初始化流程

### 在 Web 前端创建（推荐）

```
http://localhost:5173
  → 点击「+ 创建角色」
  → 填写名称 + 选择部件等级
  → 提交 → 弹出设置向导
  → 选择「Gateway Skill」或「MCP Server」
  → 一键复制 Prompt
  → 粘贴到你的 Agent 对话框
  → Agent 自动连接并开始游戏
```

### 纯命令行

```bash
# 交互式向导
python3 ember_skill.py --setup

# 或快速注册
python3 ember_skill.py --register --name "MyAgent"
```

---

## 🏗 架构

```
┌──────────────┐     WebSocket      ┌──────────────┐
│ Ember Skill  │◄──────────────────►│  Game Server │
│ (This repo)  │  tick/actions/     │  (Rule Engine)│
│              │  result/event      │              │
│     │        │                    └──────────────┘
│     │ HTTP   │
│     ▼        │
│  ┌──────┐    │
│  │ LLM  │    │
│  │ API  │    │
│  └──────┘    │
└──────────────┘
```

Skill 的职责（PRD §8.2/8.4）：
1. 管理与游戏服务器的 WebSocket 连接
2. 接收 tick 帧，提取 `messages` 转发给 LLM
3. 将 LLM 的 JSON 响应封装为 `actions` 帧发回服务器
4. 处理心跳/重连/错误

---

## 📖 相关链接

- [Ember Protocol 游戏主仓库](https://github.com/Saviorzyy/Ember-Protocol)
- [MVP PRD (完整设计文档)](https://github.com/Saviorzyy/Ember-Protocol/blob/main/docs/PRD-MVP.zh-CN.md)

## 📄 License

MIT
