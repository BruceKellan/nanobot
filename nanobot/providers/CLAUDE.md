[根目录](../../CLAUDE.md) > [nanobot](../) > **providers**

# Providers 模块

## 模块职责

LLM 提供者抽象层，统一各 LLM 服务商的接口，支持 Tool Calling、Prompt Caching、多 Provider 路由。

## 入口与启动

- 由 `cli/commands.py` 中的 `_make_provider()` 根据配置创建具体的 Provider 实例
- Provider 实例注入到 `AgentLoop`

## 对外接口

### LLMProvider 基类

```python
class LLMProvider(ABC):
    async def chat(self, messages, tools=None, model=None, max_tokens=4096, temperature=0.7, reasoning_effort=None) -> LLMResponse
    def get_default_model(self) -> str
```

### LLMResponse

```python
@dataclass
class LLMResponse:
    content: str | None
    tool_calls: list[ToolCallRequest]
    finish_reason: str
    usage: dict
    reasoning_content: str | None
    thinking_blocks: Any | None
```

### 已实现的 Provider

| Provider | 文件 | 说明 |
|----------|------|------|
| LiteLLMProvider | `litellm_provider.py` | 主力 Provider，通过 LiteLLM 支持 OpenRouter/Anthropic/OpenAI/Gemini/MiniMax 等 |
| CustomProvider | `custom_provider.py` | 直连 OpenAI 兼容 API（绕过 LiteLLM） |
| AzureOpenAIProvider | `azure_openai_provider.py` | Azure OpenAI 专用（API version 2024-10-21） |
| OpenAICodexProvider | `openai_codex_provider.py` | OpenAI Codex Responses API（OAuth 认证，SSE 流式） |
| GroqTranscriptionProvider | `transcription.py` | Groq Whisper 语音转写 |

### Provider 注册表

`registry.py` 维护已知 LLM 提供商的元数据（前缀、环境变量、缓存支持等），用于：
- 模型名到 LiteLLM 前缀的映射
- 自动检测 Gateway 类型（Ollama, LM Studio, vLLM 等）
- 模型特定参数覆盖

## 关键依赖与配置

- **LiteLLM**：核心路由库，支持 100+ LLM 后端
- **httpx**：Azure / Codex Provider 的 HTTP 客户端
- **json_repair**：修复 LLM 返回的不完整 JSON
- **oauth_cli_kit**：Codex Provider 的 OAuth 认证
- 配置项：`agents.defaults.provider`, `agents.defaults.apiKey`, `agents.defaults.apiBase`, `agents.defaults.model`

## 特性

- **Prompt Caching**：对 Anthropic 等支持的 Provider 自动注入 `cache_control`
- **Tool Call ID 规范化**：统一为 9 字符字母数字（兼容 Mistral 等严格 Provider）
- **消息净化**：自动移除非标准 key，确保跨 Provider 兼容
- **Gateway 检测**：自动识别 Ollama / LM Studio / vLLM 等本地部署

## 测试与质量

- `tests/test_azure_openai_provider.py`
- `tests/test_commands.py` 中的 Provider 相关测试

## 相关文件清单

| 文件 | 职责 |
|------|------|
| `base.py` | Provider 基类和数据类型 |
| `registry.py` | 提供商注册表 |
| `litellm_provider.py` | LiteLLM 多后端 Provider |
| `custom_provider.py` | 直连 OpenAI 兼容 API |
| `azure_openai_provider.py` | Azure OpenAI Provider |
| `openai_codex_provider.py` | OpenAI Codex Provider |
| `transcription.py` | Groq 语音转写 |

## 变更记录 (Changelog)

| 日期 | 操作 | 说明 |
|------|------|------|
| 2026-03-09 | 创建 | 初次生成 |
