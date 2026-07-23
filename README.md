# 📬 NotifyCenter - 多渠道通知中心

**一个集中管理、开箱即用的通知转发服务**，让你通过一个统一的 API，把消息推送到 Bark、Telegram、Mattermost、企业微信、PushDeer 等多个平台。适合个人自动化、Homelab、Emby/Jellyfin 媒体通知、监控告警等场景。

<p align="center">
  <img src="https://img.shields.io/badge/version-0.52-blue.svg" alt="version">
  <img src="https://img.shields.io/badge/go-1.21+-00ADD8.svg" alt="go">
  <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="license">
  <img src="https://img.shields.io/badge/docker-ready-2496ED.svg" alt="docker">
</p>

---

## ✨ v0.52 更新亮点

本次更新聚焦在**用户体验**和**功能完整性**上：

### 🎬 Emby 剧照/海报自动生成
- 播放通知消息可以直接**显示影片海报、剧照、Logo 和单集画面**
- 新增五个 Emby 模板变量：`CoverImgUrl` / `PrimaryImgUrl` / `BackdropImgUrl` / `LogoImgUrl` / `EpisodeImgUrl`
- 智能识别电影和剧集，自动使用最合适的图片（例如剧集单集自动获取该集的横版画面）
- 只需在 Emby 模板配置里填写你的 **Emby 服务器地址**，图片链接就会自动生成

### 🔔 三种新推送渠道
- **企业微信（应用消息）**：通过企业自建应用推送到员工的企业微信 App
- **企业微信（群机器人）**：更简单的方式，只需一个 Webhook Key，无需配置可信 IP
- **PushDeer**：开源自建友好的推送服务，支持官方或自建服务器

### 🔐 更贴心的密码修改流程
- 修改密码后不再突兀跳转，会显示明确的提示页 + 倒计时
- **登录成功后自动跳回你之前的页面**，不再需要重新导航

### 🎨 全新使用文档
- README 里每个渠道的配置步骤都有详细图文说明
- 补充 Bark 推送级别、Telegram Bot 创建、Mattermost 图片显示技巧、Webhook 对接示例等

### 📝 术语更清晰
- 全面统一 UI 文案中的"通道"→"路由"，避免与"渠道"混淆
  - **渠道（Channel）**：Bark、Telegram 等具体推送目标
  - **路由（Route）**：把消息分发到多个渠道的转发规则

---

## 🎯 核心特性

| 特性 | 说明 |
|------|------|
| 🚀 **一键部署** | 单个 Docker 镜像，SQLite 内嵌，无需外部数据库 |
| 📡 **7 种渠道** | Bark、Telegram、Mattermost、企业微信 x2、PushDeer、自定义 Webhook |
| 🎬 **Emby 支持** | 播放/入库事件智能识别，自动生成海报剧照 URL |
| 📝 **模板系统** | Pongo2/Jinja2 语法，支持变量、条件、循环 |
| 🔀 **路由规则** | 一次 API 调用同时推送到多个渠道 |
| 🔑 **API Key 认证** | 每个路由独立 API Key，安全隔离 |
| 📊 **可视化管理** | 内置 Web 后台，实时查看通知日志、系统日志 |
| 🌐 **时区友好** | 默认 `Asia/Shanghai`，可通过环境变量调整 |
| 🎨 **详细日志** | 每条通知记录原始数据、模板渲染结果、发送状态 |

---

## 📦 支持的通知渠道

| 渠道 | 类型标识 | 简要说明 |
|------|---------|---------|
| **Bark** | `bark` | iOS 免费推送 App，支持推送级别、角标、加密、自动复制、点击跳转、图标等 |
| **Telegram** | `telegram` | 官方 Bot API，支持个人 & 群组，HTML/Markdown 富文本 |
| **Mattermost** | `mattermost` | 开源团队协作平台，通过 Incoming Webhook 推送，支持 Markdown |
| **企业微信（应用消息）** | `wecom` | 通过企业自建应用推送到 App，支持指定接收人/部门/标签，需配置可信 IP |
| **企业微信（群机器人）** | `wecom_webhook` | 通过群机器人 Webhook 推送到企业微信群，无需可信 IP，支持 text/markdown |
| **PushDeer** | `pushdeer` | 开源推送服务，支持官方 & 自建服务器，支持 markdown/text/image |
| **自定义 Webhook** | `webhook` | 对接任意支持 Webhook 的第三方服务（如 Slack、飞书、钉钉） |

---

## 🚀 快速开始

### 使用 Docker 一键运行

```bash
docker run -d \
  --name notifycenter \
  -p 5400:5400 \
  -v $(pwd)/data:/app/data \
  -e TZ=Asia/Shanghai \
  --restart unless-stopped \
  ttt216/notifycenter:0.52
```

### 使用 Docker Compose

```yaml
version: '3.8'

services:
  notifycenter:
    image: ttt216/notifycenter:0.52
    container_name: notifycenter
    ports:
      - "5400:5400"
    volumes:
      - ./data:/app/data
    environment:
      - TZ=Asia/Shanghai
    restart: unless-stopped
```

保存为 `docker-compose.yml`，执行 `docker compose up -d` 即可。

### 访问后台

打开浏览器访问：**http://localhost:5400/admin/login**

- 默认账号：`admin`
- 默认密码：`123456`

> ⚠️ **首次登录会强制要求修改密码**，请设置一个安全的新密码（字母 + 数字组合，不少于 6 位）。

---

## 📖 使用指南

### 1️⃣ 添加通知渠道

进入 **渠道（Channels）** 菜单，点击"添加渠道"，根据不同类型填写配置。

#### 📱 Bark（iOS 推送）

1. 在手机上安装 [Bark](https://apps.apple.com/app/bark-customed-notifications/id1403753865)
2. 打开 App 首页，复制你的 **设备码（Device Key）**
3. 在 NotifyCenter 中添加 Bark 渠道，粘贴设备码
4. 可选：
   - **推送级别**：`active`（响铃）/ `timeSensitive`（勿扰模式也响）/ `passive`（不亮屏）
   - **服务器地址**：默认 `https://api.day.app`，也支持自建服务器
   - **角标数字**、**分组**、**加密**、**自动复制**、**归档** 等
5. 消息中的 `imgURL` 会作为图标，`linkURL` 作为点击跳转链接

#### 💬 Telegram

1. **创建 Bot**：与 [@BotFather](https://t.me/BotFather) 对话 → 发送 `/newbot` → 按提示操作 → 获取 Bot Token
2. **获取 Chat ID**：
   - **个人接收**：与 [@userinfobot](https://t.me/userinfobot) 对话 → 得到你的 Chat ID（正整数）
   - **群组接收**：把 Bot 拉进群 → 发一条消息 → 访问 `https://api.telegram.org/bot<TOKEN>/getUpdates` 查看 `chat.id`（负整数）
3. 在 NotifyCenter 中添加 Telegram 渠道，填写 Bot Token 和 Chat ID
4. 可选：选择 **Parse Mode** 为 `HTML` 或 `MarkdownV2` 以支持富文本

#### 🗣️ Mattermost

1. 在 Mattermost → **集成 → 传入 Webhook** → 创建 Webhook
2. 复制 Webhook URL（形如 `https://mm.example.com/hooks/xxxxx`）
3. 在 NotifyCenter 中添加 Mattermost 渠道，粘贴完整 URL
4. 可选：**用户名**（Bot 显示名）、**图标 URL**（Bot 头像）、**频道**（覆盖默认发送频道）
5. **💡 图片显示技巧**：在消息内容里用 Markdown 语法 `![]({{BackdropImgUrl}})`，Mattermost 会自动展开预览

#### 🏢 企业微信（应用消息）

1. 在 [企业微信管理后台](https://work.weixin.qq.com) 创建自建应用
2. 记录三个关键值：
   - **CorpID**（"我的企业"页面）
   - **AgentID** 和 **Secret**（应用详情页）
3. 在 NotifyCenter 中添加"企业微信（应用消息）"渠道
4. ⚠️ **重要**：需要在应用详情页的"企业可信IP"中配置服务器出口 IP，否则会返回 `60020: not allow to access from your ip` 错误
5. 可通过 `to_user` / `to_party` / `to_tag` 指定接收人/部门/标签，留空默认 `@all`

#### 🤖 企业微信（群机器人）

1. 在企业微信群中添加"群机器人"，获取 Webhook 地址
2. 在 NotifyCenter 中添加"企业微信（群机器人）"渠道
3. **Webhook Key** 支持两种格式：
   - 完整 URL：`https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx`
   - 只填 key 值：`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
4. 选择消息类型：`text`（纯文本）或 `markdown`（富文本）
5. ✅ 相比应用消息，群机器人**无需配置可信 IP**，配置更简单
6. ⚠️ 注意：只能发到企业微信群，不能发送到个人微信

#### 📮 PushDeer

1. 在 PushDeer App 中生成 **PushKey**（App 内 → "Key 管理" → "新建 Key"）
2. 在 NotifyCenter 中添加 PushDeer 渠道
3. **服务器地址**：默认 `https://api2.pushdeer.com`，也支持自建 PushDeer 服务器
4. **消息类型**：
   - `markdown`（默认）：支持 Markdown 富文本
   - `text`：纯文本
   - `image`：以图片 URL 作为消息内容

#### 🌐 自定义 Webhook

用于对接项目未内置的第三方服务（如 Slack、飞书、钉钉）。

配置项：
- **URL**：目标接口地址
- **Method**：`POST`（默认）/ `GET` / `PUT`
- **Headers**：自定义请求头（JSON）
- **Body Template**：请求体模板，支持变量替换

支持的模板变量：`{{title}}`、`{{content}}`、`{{img_url}}`、`{{link_url}}`

**示例：对接 Slack Webhook**

```json
{
  "text": "*{{title}}*\n{{content}}"
}
```

---

### 2️⃣ 配置模板

进入 **模板（Templates）** 菜单，点击"添加模板"。

**两种模板类型：**

- **general（通用模板）**：适用于普通 API 通知，配置标题和内容
- **emby（Emby 专用）**：自动识别 Emby Webhook 事件（播放开始/暂停/入库等），支持电影/剧集分别配置

**模板语法**：使用 Pongo2/Jinja2 语法，支持变量替换、条件判断、循环等。

示例：
```
🎬 {{ UserName }} 正在观看 {{ ItemName }}
📺 设备：{{ DeviceName }}
⏱️ 进度：{{ ProgressPercent }}%
```

---

### 3️⃣ 创建路由（Route）

进入 **路由（Routes）** 菜单，点击"添加路由"。

- 选择关联的**模板**
- 选择要推送到的**渠道**（可多选，一次通知同时发到多个平台）
- 保存后即可使用

---

### 4️⃣ 生成 API Key

进入 **API Key** 菜单，点击"添加 API Key"。

- 每个 API Key 绑定一个路由
- 保存后会生成唯一的 API Key，用于调用 API

---

### 5️⃣ 发送通知

**通用 API 调用**：

```bash
curl -X POST "http://your-server:5400/api/service/notify?api_key=your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "通知标题",
    "content": "通知内容",
    "img_url": "https://example.com/image.png",
    "link_url": "https://example.com"
  }'
```

**Emby Webhook 直接对接**：

在 Emby 的 Webhook 配置中填入：
```
http://your-server:5400/api/service/notify?api_key=your-api-key
```

NotifyCenter 会自动识别 Emby 的 payload，无需额外处理。

---

## 🎬 Emby 专用模板详解

NotifyCenter 完整支持 Emby 媒体服务器的 Webhook 通知，专为 Emby 优化。

### 自动事件识别

- `playback.start` / `playback.stop` / `playback.pause` / `playback.unpause` → 自动映射为 `playback.movie` 或 `playback.episode`
- `library.new` → 自动映射为 `library.new_movie` 或 `library.new_episode`

### 可用变量

| 变量 | 说明 |
|------|------|
| `Event` | 事件类型 |
| `UserName` | 用户名 |
| `ItemName` | 媒体名称 |
| `ItemType` | 媒体类型（Movie / Episode） |
| `ItemYear` | 发行年份 |
| `ItemOverview` | 简介（自动截断至 200 字符） |
| `SeriesName` | 剧集名称（仅剧集） |
| `Season` / `Episode` | 季数 / 集数 |
| `DeviceName` / `Client` / `ClientIp` | 播放设备信息 |
| `ServerName` / `ServerVersion` | 服务器信息 |
| `DateLocal` | 本地时间（自动转换时区） |
| `CoverImgUrl` / `PrimaryImgUrl` | 海报封面（竖版；剧集为整剧海报） |
| `BackdropImgUrl` | 剧照/背景图（横版） |
| `LogoImgUrl` | 影片 Logo 图 |
| `EpisodeImgUrl` | **单集画面**（仅剧集播放时有值） |
| `ProgressMinutes` / `TotalMinutes` / `ProgressPercent` | 播放进度 |

### 图片变量使用说明

> **需要配置 Emby 服务器地址**：在模板管理 → 编辑 Emby 模板 → 填写 **Emby 服务器地址**（如 `https://emby.example.com:8086`），系统会自动生成图片 URL。不填写则四个图片变量均为空字符串。

### 示例：剧集播放模板

```
🎬 {{ UserName }} 正在观看《{{ SeriesName }}》
📺 第 {{ Season }} 季第 {{ Episode }} 集 - {{ ItemName }}
⏱️ 进度：{{ ProgressPercent }}%

![]({{ EpisodeImgUrl }})
```

---

## ⚙️ 环境变量

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `TZ` | `Asia/Shanghai` | 时区配置 |
| `DATA_PATH` | 容器内 `/app/data` | 数据存储路径 |
| `ADMIN_SESSION_KEY` | 内置默认值 | 管理后台会话安全密钥（建议自行设置） |
| `APP_VERSION` | 从 VERSION 文件读取 | 应用版本号（一般无需设置） |

---

## 💾 数据持久化

- 数据存储在 SQLite 数据库中，位于容器内 `/app/data/notifycenter.db`
- 通过挂载 `./data:/app/data` 可以持久化数据
- **升级版本时数据完全兼容**，只需替换镜像重启即可

### 从旧版本升级

1. 停止旧容器
2. 备份 `data/notifycenter.db` 文件（推荐）
3. 使用 v0.52 镜像创建新容器，挂载相同的 `data` 目录
4. 数据库结构完全兼容，无需迁移

---

## 🎯 典型应用场景

| 场景 | 说明 |
|------|------|
| 📺 **Emby / Jellyfin 播放通知** | 家人开始观看/新入库剧集时推送到手机 |
| 🖥️ **监控告警** | 服务器 CPU/内存告警推送到多个渠道 |
| 🤖 **自动化脚本通知** | Python/Shell 脚本执行结果推送 |
| 📦 **下载完成提醒** | qBittorrent/Aria2 下载完成通知 |
| 🔒 **登录/安全告警** | 检测到异常登录立即推送 |
| 📊 **日报/周报推送** | 定时任务生成的统计报告 |

---

## ❓ 常见问题

### 如何获取服务器公网 IP？

```bash
curl ifconfig.me
```

用于企业微信应用消息的"企业可信IP"配置。

### 忘记管理员密码怎么办？

删除数据库文件重新初始化：

```bash
docker stop notifycenter
rm data/notifycenter.db
docker start notifycenter
```

> ⚠️ 该操作会丢失所有配置数据，请谨慎操作。

### 如何测试渠道是否配置正确？

在 **渠道管理** 页面，每个渠道右侧都有 **测试** 按钮，点击后可以填写测试标题和内容进行发送，会显示详细的响应信息。

### 支持 arm64 / Apple Silicon 吗？

当前发布版本为 amd64 架构。如需 arm64，可以拉取源码本地构建。

---

## 🛠️ 技术栈

- **后端**：Go 1.21+ / Gin / GORM / SQLite
- **模板引擎**：Pongo2（Jinja2 兼容）
- **前端**：原生 HTML + Tailwind CSS + Vanilla JS
- **容器**：Alpine Linux 精简镜像

---

## 📄 更新日志

### v0.52（当前版本）
- 🎨 **界面术语统一**：所有 UI 文案中的"通道"改为"路由"，避免与"渠道（Channel）"混淆
- 🔐 **密码修改体验优化**：修改成功后显示提示页 + 倒计时，登录后自动跳回原页面
- 📖 **使用文档大幅完善**：Bark、Telegram、Mattermost、Webhook 配置步骤全面补齐，附典型示例
- 🚀 **性能优化**：版本号读取从每次请求改为启动加载，减少文件 IO
- 🧹 **代码质量提升**：清理冗余代码和临时文件，项目结构更清晰

### v0.51
- 🎬 **Emby 图片链接自动生成**：新增 `CoverImgUrl` / `PrimaryImgUrl` / `BackdropImgUrl` / `LogoImgUrl` / `EpisodeImgUrl` 五个模板变量
- 📸 **单集画面独立变量**：新增 `EpisodeImgUrl`，剧集播放通知可显示单集自身画面
- 🎯 **智能回退**：剧集单集自动使用父级（Series）的 Backdrop / Logo 图片
- ⚙️ **模板配置扩展**：Emby 模板可配置服务器地址，图片 URL 自动拼接

### v0.5
- 🆕 新增**企业微信（应用消息）**渠道
- 🆕 新增**企业微信（群机器人）**渠道
- 🆕 新增 **PushDeer** 渠道
- 🔧 修复渠道配置存储的多个问题
- 📊 Demo 初始化数据完善

### v0.4
- 🚀 项目从 Python (FastAPI) 全面重写为 Go (Gin)
- ⚡ 启动速度、性能、部署体积显著优化
- 🎨 保持所有功能、页面、API 完全兼容

---

## 📄 许可证

MIT License

---

## 🙋 反馈与建议

如果你在使用中遇到问题、有新功能建议或想贡献代码，欢迎提交 Issue 或 Pull Request。

**祝使用愉快！** 🎉
