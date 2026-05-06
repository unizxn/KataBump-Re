# Katabump Server Auto-Renewal Tool

> 用于自动续期 Katabump 服务器的自动化脚本。利用 Playwright + CDP (Chrome DevTools Protocol) 模拟用户操作，有效绕过 Cloudflare Turnstile 与 ALTCHA 验证码，确保持续的服务器服务。

通过 **GitHub Actions** 云端定时运行。

---

## ✨ 特性

- **智能过盾**：通过 CDP 协议模拟真实鼠标轨迹和点击行为，结合屏幕坐标伪造，高成功率绕过 Cloudflare Turnstile 与 ALTCHA。
- **自动重试**：内置严格的验证重试机制，如果验证失败会自动重启验证流程。
- **多用户支持**：支持配置多个账号批量续期。
- **SOCKS5 代理支持**：通过 gost 将 SOCKS5 转为本地的 HTTP 代理，兼容 Chrome 的代理机制。
- **Telegram 通知**：续期结果（成功/失败/跳过/异常）自动推送到 Telegram，附带截图。
- **日志脱敏**：公开仓库运行时代理地址与出口 IP 自动隐藏，防止节点信息泄露。

---

## 🚀 快速开始

### 1. Fork 本仓库

点击右上角 **Fork** 到你的 GitHub 账号。

### 2. 配置 Secrets

进入仓库 → `Settings` → `Secrets and variables` → `Actions` → `New repository secret`。

#### 必填

| Secret Name | 格式示例 | 说明 |
|---|---|---|
| `USERS_JSON` | `[{"username":"a@b.com","password":"pwd"}]` | 账号密码 JSON 数组（尽量压缩为一行） |

#### 代理（可选但强烈建议）

| Secret Name | 格式示例 | 说明 |
|---|---|---|
| `SOCKS5_PROXY` | `socks5://user:pass@host:port` | SOCKS5 代理（带认证或不带认证） |

> 💡 本项目通过 gost 将 SOCKS5 转成本地 HTTP 代理 (`http://127.0.0.1:8080`)，因为 Chrome 原生不支持 SOCKS5 认证。

#### Telegram 通知（可选）

| Secret Name | 获取方式 | 说明 |
|---|---|---|
| `TG_BOT_TOKEN` | `@BotFather` | Telegram Bot Token |
| `TG_CHAT_ID` | `@userinfobot` 或 API | 用户 ID 或群组 ID |

### 3. 启用 Actions

进入 **Actions** 页面，点击左侧工作流名称，然后点击 **Enable workflow**。

工作流默认 **每天北京时间 05:00 (UTC 21:00)** 自动运行，也可手动点击 **Run workflow** 测试。

---

## 🌐 代理配置详解

### SOCKS5 代理

```
socks5://127.0.0.1:1080
socks5://username:password@127.0.0.1:1080
```

### 代理工作原理

1. GitHub Actions 运行环境安装 **gost**（v2.11.5）。
2. gost 在本地 `127.0.0.1:8080` 启动 HTTP 入站代理，后端连接你的 SOCKS5 代理。
3. Chrome 浏览器通过 `--proxy-server=http://127.0.0.1:8080` 走本地 HTTP 代理。
4. 运行前自动验证代理连通性（`https://1.1.1.1`）。
5. **日志中代理服务器地址自动脱敏显示为 `***`**，防止公开仓库泄露节点信息。

> ⚠️ 为什么不直接传 SOCKS5 给 Chrome？因为 Chrome 的 `--proxy-server` 对 SOCKS5 认证支持极差，会报 `ERR_NO_SUPPORTED_PROXIES`。通过 gost 中转是稳定方案。

---

## 📸 运行结果与截图

- **运行日志**：在 Actions 中的 `Run Renew Script` 步骤查看。
- **截图留存**：每次运行（无论成功与否），通过 `Upload Screenshots` 步骤自动上传。

  - 在 Workflow 运行详情页的 **Artifacts** 区域下载 `screenshots` 压缩包。
  - 每个账号对应一张截图（`username.png`），方便确认状态。

---

## 🛠️ 项目结构

```
.
├── action_renew.js              # 主程序脚本（适配 GitHub Actions Linux/xvfb）
├── package.json                 # 依赖配置
├── .github/workflows/renew.yml  # GitHub Actions 定时任务配置
└── README.md                    # 本文件
```

---

## ⚠️ 常见问题

### Q1: 如何配置多账号？

`USERS_JSON` 支持多个对象：

```json
[
  {"username": "user1@example.com", "password": "pass1"},
  {"username": "user2@example.com", "password": "pass2"}
]
```

### Q2: 代理验证通过但页面打不开？

检查 `SOCKS5_PROXY` 格式是否为标准 URI：`socks5://user:pass@host:port`。如果节点不支持用户名密码认证，尝试去掉认证部分。

### Q3: Telegram 没收到通知？

- 确认 `TG_BOT_TOKEN` 和 `TG_CHAT_ID` 已配置。
- 确认 Bot 已发送 `/start` 激活对话。
- 如果是群组，确认 Bot 已被加入群组。
- 即使脚本异常崩溃，最新版本也会推送错误通知。

### Q4: 如何获取 Telegram Chat ID？

1. 在 Telegram 搜索 `@userinfobot`，发送 `/start` 获取个人 ID。
2. 群组 ID：将 Bot 加入群组后发送消息，访问 `https://api.telegram.org/bot<Token>/getUpdates` 查看 `chat.id`。

### Q5: 日志里为什么看不到代理 IP？

这是有意设计的。公开仓库的 Actions 日志对所有人可见，脚本已自动将代理服务器地址和出口 IP 脱敏为 `***`，防止节点信息泄露。

---

## 🙏 特别鸣谢

- `550530/katabump-renew` — 原项目
- `playwright` / `puppeteer-extra-plugin-stealth` — 浏览器自动化框架
- `ginuerzh/gost` — 代理转发工具

---

## 📜 License

MIT License
