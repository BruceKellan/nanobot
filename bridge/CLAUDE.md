[根目录](../CLAUDE.md) > **bridge**

# WhatsApp Bridge 模块

## 模块职责

Node.js WhatsApp 桥接器，使用 Baileys 库实现 WhatsApp Web 协议，通过 WebSocket 与 Python 后端通信。

## 入口与启动

- 入口文件：`src/index.ts`
- 启动命令：`npm run build && npm start`
- 环境变量：
  - `BRIDGE_PORT` - WebSocket 端口（默认 3001）
  - `AUTH_DIR` - 认证数据目录（默认 `~/.nanobot/whatsapp-auth`）
  - `BRIDGE_TOKEN` - 可选的认证令牌

## 对外接口

WebSocket 协议消息格式：

- **入站**（Bridge -> Python）：
  - `{type: "message", sender, pn, content, id, media, isGroup, timestamp}`
  - `{type: "status", status: "connected"|"disconnected"}`
  - `{type: "qr", ...}` - QR 码认证
  - `{type: "error", error: "..."}`

- **出站**（Python -> Bridge）：
  - `{type: "send", to, text}`
  - `{type: "auth", token}` - 认证

## 关键依赖与配置

- `@whiskeysockets/baileys` 7.0.0-rc.9 - WhatsApp Web 协议
- `ws` - WebSocket 服务器
- `qrcode-terminal` - 终端 QR 码显示
- Node.js >= 20.0.0
- TypeScript 5.4+

## 安全特性

- 绑定 `127.0.0.1`（仅本地访问）
- 可选 `bridgeToken` 共享密钥认证
- 认证数据目录需设置 `chmod 700`

## 相关文件清单

| 文件 | 职责 |
|------|------|
| `src/index.ts` | 入口，启动 BridgeServer |
| `src/server.ts` | WebSocket 服务器和 Baileys 集成 |
| `package.json` | 依赖和脚本 |
| `tsconfig.json` | TypeScript 配置 |

## 变更记录 (Changelog)

| 日期 | 操作 | 说明 |
|------|------|------|
| 2026-03-09 | 创建 | 初次生成 |
