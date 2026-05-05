# Katabump Server Auto-Renewal Tool

> 用于自动续期 Katabump 服务器的自动化脚本。利用 Playwright + CDP (Chrome DevTools Protocol) 模拟用户操作，有效绕过 Cloudflare Turnstile 与 ALTCHA 验证码，确保持续的服务器服务。

通过 **GitHub Actions** 云端定时运行。

---

## ✨ 特性

- **智能过盾**：通过 CDP 协议模拟真实鼠标轨迹和点击行为，结合屏幕坐标伪造，高成功率绕过 Cloudflare Turnstile 与 ALTCHA。
- **自动重试**：内置严格的验证重试机制，如果验证失败会自动重启验证流程。
- **多用户支持**：支持配置多个账号批量续期。
- **双协议代理**：支持 `HTTP` 与 `SOCKS5` 代理（带认证或不带认证）。
- **Telegram 通知**：续期结果（成功/失败/跳过）自动推送到 Telegram，附带截图。
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

#### 代理（二选一，可选）

| Secret Name | 格式示例 | 说明 |
|---|---|---|
| `HTTP_PROXY` | `http://user:pass@host:port` | HTTP/HTTPS 代理 |
| `SOCKS5_PROXY` | `socks5://user:pass@host:port` | SOCKS5 代理 |

> ⚠️ 优先级：`HTTP_PROXY` > `SOCKS5_PROXY`。两者同时配置时优先使用 HTTP 代理。不配置则直连。

#### Telegram 通知（可选）

| Secret Name | 获取方式 | 说明 |
|---|---|---|
| `TG_BOT_TOKEN` | `@BotFather` | Telegram Bot Token |
| `TG_CHAT_ID` | `@userinfobot` 或 API | 用户 ID 或群组 ID |

### 3. 启用 Actions

进入 **Actions** 页面，点击左侧工作流名称，然后点击 **Enable workflow**。

工作流默认 **每天北京时间 08:00 (UTC 00:00)** 自动运行，也可手动点击 **Run workflow** 测试。

---

## 🌐 代理配置详解

### HTTP 代理

```
http://127.0.0.1:8080
http://username:password@127.0.0.1:8080
```

### SOCKS5 代理

```
socks5://127.0.0.1:1080
socks5://username:password@127.0.0.1:1080
```

### 代理工作原理

1. 脚本读取 `HTTP_PROXY` 或 `SOCKS5_PROXY`。
2. 启动 Chrome 时注入 `--proxy-server` 参数。
3. SOCKS5 认证由 Chrome 原生支持；HTTP 代理认证通过 Playwright 请求拦截附加 `Proxy-Authorization` 头。
4. 运行前自动验证代理连通性（`https://1.1.1.1`）。
5. **日志中代理服务器地址自动脱敏显示为 `***`**，防止公开仓库泄露节点信息。

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
├── package.json                 # 依赖配置（含 socks-proxy-agent）
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

### Q2: HTTP_PROXY 和 SOCKS5_PROXY 同时配置会怎样？

优先使用 `HTTP_PROXY`。如需使用 SOCKS5，请暂时清空 `HTTP_PROXY`。

### Q3: 代理验证失败怎么办？

- 检查 Secret 格式是否正确（注意 `socks5://` 协议头）。
- 确认代理节点在 GitHub Actions 出口网络可达。
- 查看 Actions 日志中 `[代理]` 开头的输出。

### Q4: 如何获取 Telegram Chat ID？

1. 在 Telegram 搜索 `@userinfobot`，发送 `/start` 获取个人 ID。
2. 群组 ID：将 Bot 加入群组后发送消息，访问 `https://api.telegram.org/bot<Token>/getUpdates` 查看 `chat.id`。

### Q5: 日志里为什么看不到代理 IP？

这是有意设计的。公开仓库的 Actions 日志对所有人可见，脚本已自动将代理服务器地址和出口 IP 脱敏为 `***`，防止节点信息泄露。

---

## 🙏 特别鸣谢

- `550530/katabump-renew` — 原项目
- `playwright` / `puppeteer-extra-plugin-stealth` — 浏览器自动化框架
- `socks-proxy-agent` — SOCKS5 代理支持

---

## 📜 License

MIT License
