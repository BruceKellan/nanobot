[根目录](../../../CLAUDE.md) > [nanobot](../../) > [agent](../) > **tools**

# Agent Tools 模块

## 模块职责

工具注册中心和所有内置工具的实现。Agent 通过 ToolRegistry 查找并执行工具，LLM 通过 function calling 协议调用。

## 入口与启动

- **ToolRegistry** (`registry.py`)：工具注册与查找中心
  - `register(tool)` - 注册一个 Tool 实例
  - `get_tool(name)` - 按名称查找
  - `get_definitions()` - 返回所有工具的 OpenAI 格式定义

## 对外接口

### Tool 基类

```python
class Tool(ABC):
    @property
    def name(self) -> str
    @property
    def description(self) -> str
    @property
    def parameters(self) -> dict[str, Any]
    async def execute(self, **kwargs) -> str
```

### 内置工具列表

| 工具名 | 文件 | 功能 |
|--------|------|------|
| `exec` | `shell.py` | Shell 命令执行（带危险命令过滤） |
| `read_file` | `filesystem.py` | 读取文件内容（128KB 上限） |
| `write_file` | `filesystem.py` | 写入文件 |
| `edit_file` | `filesystem.py` | 文本替换编辑（带模糊匹配提示） |
| `list_dir` | `filesystem.py` | 列出目录内容 |
| `web_search` | `web.py` | Brave Search API 网页搜索 |
| `web_fetch` | `web.py` | 获取并提取 URL 内容（Readability） |
| `message` | `message.py` | 向用户发送消息 |
| `spawn` | `spawn.py` | 创建后台子代理 |
| `cron` | `cron.py` | 定时任务管理（add/list/remove） |
| `mcp_*` | `mcp.py` | MCP 协议工具包装器（stdio/SSE/streamableHttp） |

## 关键依赖与配置

- `exec` 工具：`deny_patterns` 危险命令黑名单，`restrict_to_workspace` 路径限制
- `web_search`：需要 `BRAVE_API_KEY`
- `web_fetch`：使用 `readability-lxml` 提取可读内容
- `mcp`：支持 stdio、SSE、streamableHttp 三种传输协议

## 安全特性

- Shell 工具 (`exec`)：正则黑名单过滤 `rm -rf`、fork bomb、disk format 等
- 文件工具：路径遍历保护 (`_resolve_path`)，可选的 `allowed_dir` 限制
- Web 工具：URL 验证（仅 http/https），重定向限制 (5 次)
- MCP 工具：超时保护 (`tool_timeout`)

## 测试与质量

- `tests/test_tool_validation.py`
- `tests/test_mcp_tool.py`
- `tests/test_message_tool.py`
- `tests/test_message_tool_suppress.py`

## 变更记录 (Changelog)

| 日期 | 操作 | 说明 |
|------|------|------|
| 2026-03-09 | 创建 | 初次生成 |
