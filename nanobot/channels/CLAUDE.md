[根目录](../../CLAUDE.md) > [nanobot](../) > **channels**

# Channels 模块

## 模块职责

消息渠道适配层，负责将各即时通讯平台的消息协议统一为 nanobot 的内部消息格式 (InboundMessage / OutboundMessage)，通过 MessageBus 与 Agent 交互。

## 入口与启动

- **ChannelManager** (`manager.py`)：统一管理所有渠道的启停
  - `start_all()` - 并发启动所有已启用的渠道
  - `stop_all()` - 停止所有渠道
- 由 `gateway` CLI 命令调用

## 对外接口

### BaseChannel

```python
class BaseChannel(ABC):
    name: str
    async def start(self) -> None
    async def stop(self) -> None
    async def send(self, msg: OutboundMessage) -> None
    def is_allowed(self, sender_id: str) -> bool
```

### 已实现的渠道

| 渠道 | 文件 | 协议 | 特点 |
|------|------|------|------|
| Telegram | `telegram.py` | Long Polling | Markdown 转 HTML，流式模拟发送，语音转写 |
| Discord | `discord.py` | Gateway WebSocket | 群组 @mention 策略，文件附件，重连机制 |
| WhatsApp | `whatsapp.py` | WebSocket (Node.js Bridge) | 通过 bridge/ 桥接 Baileys |
| 飞书 (Feishu) | `feishu.py` | WebSocket | 富文本消息，表情回应 |
| 钉钉 (DingTalk) | `dingtalk.py` | Stream | AppKey/AppSecret 认证 |
| Matrix | `matrix.py` | Matrix Client-Server | E2EE 加密支持 |
| Slack | `slack.py` | Socket Mode | Slack App 集成 |
| QQ | `qq.py` | - | QQ 机器人 |
| Email | `email.py` | IMAP + SMTP | 邮箱收发 |
| MoChat | `mochat.py` | - | 内部聊天协议 |

## 关键依赖与配置

- 每个渠道都需要在 `~/.nanobot/config.json` 中配置 `channels.<name>.enabled` 和 `channels.<name>.token`
- `allowFrom` 白名单控制访问权限（v0.1.4.post4 起空列表默认拒绝所有）
- Telegram 支持 HTTP/SOCKS5 代理 (`proxy` 配置项)
- Discord 支持 `groupPolicy`：`mention`（需 @）或 `open`（所有消息）

## 数据模型

- `InboundMessage`：sender_id, channel, chat_id, content, media, metadata
- `OutboundMessage`：channel, chat_id, content, media, metadata
- 消息通过 `MessageBus` 发布/订阅

## 测试与质量

- `tests/test_base_channel.py`
- `tests/test_telegram_channel.py`
- `tests/test_dingtalk_channel.py`
- `tests/test_feishu_post_content.py`
- `tests/test_feishu_table_split.py`
- `tests/test_email_channel.py`
- `tests/test_matrix_channel.py`
- `tests/test_qq_channel.py`

## 常见问题 (FAQ)

- **如何新增渠道？** 继承 `BaseChannel`，实现 `start/stop/send`，在 `ChannelManager` 中注册
- **消息权限控制？** 通过 `is_allowed(sender_id)` 和 `allowFrom` 配置
- **消息过长怎么办？** 使用 `split_message()` 工具函数分割（各渠道限制不同：Telegram 4000, Discord 2000）

## 相关文件清单

| 文件 | 职责 |
|------|------|
| `base.py` | 渠道基类 |
| `manager.py` | 渠道管理器 |
| `telegram.py` | Telegram 适配 |
| `discord.py` | Discord 适配 |
| `whatsapp.py` | WhatsApp 适配 |
| `feishu.py` | 飞书适配 |
| `dingtalk.py` | 钉钉适配 |
| `matrix.py` | Matrix 适配 |
| `slack.py` | Slack 适配 |
| `qq.py` | QQ 适配 |
| `email.py` | Email 适配 |
| `mochat.py` | MoChat 适配 |

## 变更记录 (Changelog)

| 日期 | 操作 | 说明 |
|------|------|------|
| 2026-03-09 | 创建 | 初次生成 |
