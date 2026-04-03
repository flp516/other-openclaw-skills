---
name: wecom-bot-setup
description: 企业微信智能机器人接入 OpenClaw 的配置指南。当用户需要配置企业微信智能机器人连接到 OpenClaw 时使用此技能。触发词：企业微信机器人配置、wecom bot setup、企业微信接入、配置企业微信channel。
---

# 企业微信智能机器人接入 OpenClaw 配置指南

本技能指导用户将企业微信智能机器人接入 OpenClaw，实现通过企业微信与 AI 助手对话。

## 前置条件

- 企业微信管理员权限
- OpenClaw 已安装并运行（版本 >= 2026.3.28）

## 配置步骤

### 第一步：在企业微信后台创建智能机器人

1. 登录 [企业微信管理后台](https://work.weixin.qq.com/wework_admin/loginpage_wx)
2. 点击顶部菜单 **「应用管理」**
3. 点击 **「创建应用」**
4. 选择 **「智能机器人」**（注意：不是自建应用）
5. 填写机器人名称、描述、头像
6. **选择「API模式」**（关键步骤）
7. 点击创建

### 第二步：获取机器人凭证

创建成功后，在机器人详情页获取：
- **Bot ID**（类似 `aibbaASmjPf24FGucaXD1FL-YHc3zOSbcWI`）
- **安全密钥**（类似 `otrlrVTviRiDuW1m4adtio8RAvbzMsHPGtz5tymvVVa`）

**重要**：妥善保管密钥，不要泄露。

### 第三步：配置机器人可见范围

1. 在机器人详情页，找到 **「可见范围」** 设置
2. 添加需要使用机器人的成员或部门
3. 建议先添加自己进行测试

### 第四步：安装企业微信插件

#### 方式一：使用官方 CLI 工具（推荐）

```bash
npx -y @wecom/wecom-openclaw-cli install --force
```

此命令会自动：
- 安装最新版插件
- 安装所需依赖
- 禁用冲突插件

#### 方式二：手动安装

```bash
openclaw plugins install @wecom/wecom-openclaw-plugin
```

**注意**：插件包名是 `wecom-openclaw-plugin`（不是 `wecom-openclash-plugin`）

### 第五步：配置 OpenClaw

#### 方式一：使用 CLI 命令（推荐）

```bash
# 设置 Bot ID
openclaw config set channels.wecom.botId "你的Bot ID"

# 设置密钥
openclaw config set channels.wecom.secret "你的安全密钥"

# 启用通道
openclaw config set channels.wecom.enabled true

# 设置连接模式（WebSocket 模式）
openclaw config set channels.wecom.connectionMode "websocket"

# 设置私信访问策略（配对模式）
openclaw config set channels.wecom.dmPolicy "pairing"

# 设置群组访问策略（白名单模式）
openclaw config set channels.wecom.groupPolicy "allowlist"
```

#### 方式二：手动编辑配置文件

编辑 `~/.openclaw/openclaw.json`，在 `channels` 下添加：

```json
"wecom": {
  "enabled": true,
  "botId": "你的Bot ID",
  "secret": "你的安全密钥",
  "connectionMode": "websocket",
  "dmPolicy": "pairing",
  "groupPolicy": "allowlist"
}
```

#### 配置项说明

| 配置项 | 说明 | 可选值 | 默认值 |
|--------|------|--------|--------|
| `enabled` | 是否启用 | `true` / `false` | `false` |
| `botId` | 机器人 ID | - | - |
| `secret` | 安全密钥 | - | - |
| `connectionMode` | 连接模式 | `websocket` / `webhook` | `websocket` |
| `dmPolicy` | 私信访问策略 | `pairing` / `open` / `allowlist` / `disabled` | `open` |
| `groupPolicy` | 群组访问策略 | `open` / `allowlist` / `disabled` | `open` |

### 第六步：重启 OpenClaw Gateway

```bash
openclaw gateway restart
```

检查状态：
```bash
openclaw status
```

确认 `企业微信` channel 显示 `OK` 或 `SETUP` 状态。

**注意**：WebSocket 模式下显示 `SETUP: no token` 是正常的，表示连接已建立但还未收到消息。

### 第七步：完成配对

1. 打开企业微信 App
2. 在「工作台」或搜索框找到你的智能机器人
3. 给机器人发送任意消息
4. 机器人会返回配对提示，包含：
   - 你的企业微信用户ID
   - 配对码（如 `ESJAL6K6`）
5. 在 OpenClaw 终端执行：
   ```bash
   openclaw pairing approve wecom <配对码>
   ```

### 第八步：验证连接

配对成功后，在企业微信中给机器人发送消息测试：
- 发送 "你好"
- 确认收到 AI 助手的回复

## 连接模式说明

### WebSocket 模式（推荐）

- **优点**：配置简单，支持流式回复
- **配置**：只需 `botId` + `secret`
- **适用**：大多数场景

### Webhook 模式

- **优点**：更灵活的消息处理
- **配置**：需要 `token` + `encodingAESKey`
- **适用**：需要自定义消息处理的场景

## 常见问题

### 插件安装失败

- 使用 `--force` 参数强制安装：`npx -y @wecom/wecom-openclaw-cli install --force`
- 检查 OpenClaw 版本是否 >= 2026.3.28

### 找不到智能机器人

- 确认已在「可见范围」中添加自己
- 在企业微信 App 顶部搜索框搜索机器人名称
- 检查机器人是否已发布

### 配对失败

- 确认 Bot ID 和密钥正确
- 检查 OpenClaw gateway 是否正常运行
- 查看 `openclaw status` 确认 channel 状态

### 消息无响应

- 确认配对已成功
- 检查 OpenClaw 日志：`openclaw logs`
- 确认 WebSocket 连接已建立（日志中应有 `WebSocket connection established`）

### 状态显示 "no token"

- WebSocket 模式下这是正常的，不需要 token
- 只要日志显示 `Authentication successful` 即表示连接成功

## 相关资源

- [企业微信开放平台文档](https://developer.work.weixin.qq.com/document/)
- [企业微信智能机器人官方文档](https://open.work.weixin.qq.com/help?doc_id=21657)
- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [企业微信 OpenClaw 插件完整接入指引](https://doc.weixin.qq.com/doc/w3_AFYA1wY6ACoCNRxfnyGRJQaSa6jjJ?scode=AJEAIQdfAAo0RJmzxLAFYA1wY6ACo)
