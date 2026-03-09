[根目录](../../CLAUDE.md) > [nanobot](../) > **agent**

# Agent 模块

## 模块职责

Agent 模块是 nanobot 的核心处理引擎，负责：

- 接收用户消息并构建 LLM 上下文（系统提示 + 对话历史 + 记忆 + 技能）
- 调用 LLM 获取响应
- 解析并执行 Tool Call（工具调用循环）
- 管理子代理 (Subagent) 的后台任务执行
- 管理长期记忆的存取与合并

## 入口与启动

- **AgentLoop** (`loop.py`)：核心循环类，由 CLI 的 `agent`/`gateway`/`chat` 命令创建
  - `process_message(msg)` - 处理来自 MessageBus 的消息
  - `process_direct(content)` - 直接处理文本（CLI 模式）
  - 内部执行 Tool Call 循环（最多 `max_iterations` 次）

## 对外接口

### AgentLoop

```python
class AgentLoop:
    def __init__(self, bus, provider, workspace, model, max_iterations=40, ...)
    async def process_message(self, msg: InboundMessage) -> None
    async def process_direct(self, content: str, ...) -> str
    async def close_mcp(self) -> None
```

### ContextBuilder

```python
class ContextBuilder:
    def build_system_prompt(self, skill_names=None) -> str
    def build_messages(self, session, channel, chat_id, content, media) -> list[dict]
```

### MemoryStore

```python
class MemoryStore:
    def get_memory_context(self) -> str
    async def consolidate(self, session, provider, model) -> None
```

### SubagentManager

```python
class SubagentManager:
    async def spawn(self, task, label, origin_channel, origin_chat_id, session_key) -> str
```

## 关键依赖与配置

- **内部依赖**：`bus.events`, `bus.queue`, `config.schema`, `session.manager`, `providers.base`
- **外部依赖**：loguru, asyncio
- 配置项：`agents.defaults.model`, `agents.defaults.maxIterations`, `agents.defaults.temperature`

## 子模块：tools/

工具注册中心与所有内置工具，详见 [tools/CLAUDE.md](./tools/CLAUDE.md)。

## 数据模型

- `Session`：包含 messages 列表 (追加写入)，last_consolidated 标记
- 上下文由 `ContextBuilder` 从 AGENTS.md, SOUL.md, USER.md, TOOLS.md, MEMORY.md, HISTORY.md, SKILL.md 等文件动态组装

## 测试与质量

- `tests/test_loop_save_turn.py` - Agent 循环保存测试
- `tests/test_task_cancel.py` - 任务取消测试
- `tests/test_context_prompt_cache.py` - 上下文缓存测试
- `tests/test_memory_consolidation_types.py` - 记忆合并测试
- `tests/test_consolidate_offset.py` - 合并偏移测试

## 相关文件清单

| 文件 | 职责 |
|------|------|
| `loop.py` | 核心 Agent 处理循环 |
| `context.py` | 系统提示与消息上下文构建 |
| `memory.py` | 长期记忆存储与合并 |
| `skills.py` | 技能加载器 |
| `subagent.py` | 子代理管理 |
| `tools/` | 工具子模块 |

## 变更记录 (Changelog)

| 日期 | 操作 | 说明 |
|------|------|------|
| 2026-03-09 | 创建 | 初次生成 |
