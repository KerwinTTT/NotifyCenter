# 📢 NotifyCenter - 多渠道通知中心

一个功能完整的多渠道通知转发服务，支持 Bark、Telegram、Mattermost、Webhook 等多种通知方式。通过统一的 API 接口，快速将通知推送到多个渠道，支持自定义通知模板和路由规则。

---

## 🎯 核心功能

### 📬 支持的通知渠道

| 渠道 | 描述 |
|------|------|
| 🔔 **Bark** | iOS 设备即时推送 |
| 🤖 **Telegram** | Telegram Bot 消息 |
| 💬 **Mattermost** | Mattermost 团队协作消息 |
| 🌐 **Webhook** | 自定义 Webhook |

###  核心特性

- ✅ **统一 API 接口**：简单 HTTP 请求即可发送通知
- ✅ **模板系统**：支持通用模板和 Emby 专用模板
- ✅ **路由管理**：灵活配置 API Key 与模板、渠道的关联
- ✅ **API Key 管理**：支持生成、启用/禁用、删除 API Key
- ✅ **完整日志**：推送日志和后端日志，支持查看详情
- ✅ **Web 管理后台**：友好的用户界面，轻松配置所有功能
- ✅ **安全机制**：自动生成随机会话密钥

---

## 🚀 快速开始

### 使用 Docker 运行（推荐）

```bash
docker run -d \
  --name notifycenter \
  -p 5400:5400 \
  -v ./data:/data \
  -e TZ=Asia/Shanghai \
  --restart unless-stopped \
  ttt216/notifycenter:latest
```

### 使用 Docker Compose 运行

```yaml
version: '3.8'

services:
  notifycenter:
    image: ttt216/notifycenter:latest
    container_name: notifycenter
    ports:
      - "5400:5400"
    volumes:
      - ./data:/data
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - DATA_PATH=/data
      # - ADMIN_SESSION_KEY=your-custom-secret-key-here  # 可选，不设置时自动生成
      - PORT=5400
```

---

## 🌐 访问地址

- **管理后台**：http://localhost:5400/admin/login
- **默认账号**：`admin` / `123456`
- **API 文档**：http://localhost:5400/docs

⚠️ 首次登录请立即修改密码！

---

## 📖 使用说明

### 1️⃣ 登录系统

1. 访问 http://localhost:5400/admin/login
2. 使用默认账号登录
3. 首次登录会强制要求修改密码

### 2️⃣ 配置通知渠道

#### Bark 配置：
1. 进入 **渠道（Channels）** 菜单
2. 点击 **添加渠道（Add Channel）** 选择 **Bark**
3. 填写配置：
   - **设备码（Device Key）**：从 Bark App 获取的设备码
   - 可选配置：服务器地址、推送级别、角标等
4. 保存并测试

#### Telegram 配置：
1. 创建 Telegram Bot（从 @BotFather 获取 Token
2. 获取 Chat ID（使用 @userinfobot
3. 在系统中添加 Telegram 渠道
4. 填写 Bot Token 和 Chat ID

#### Mattermost 配置：
1. 在 Mattermost 中创建 Incoming Webhook
2. 获取 Server URL 和 Webhook Key
3. 在系统中添加 Mattermost 渠道

#### Webhook 配置：
1. 添加自定义 Webhook 渠道
2. 支持自定义请求方法、请求头、请求体模板
3. 支持变量：`{{title}}`、`{{content}}`、`{{img_url}}`、`{{link_url}}`

### 3️⃣ 设置模板

1. 进入 **模板（Templates）** 菜单
2. 点击 **添加模板（Add Template）**
3. 配置模板标题和内容

### 4️⃣ 创建路由

1. 进入 **路由（Routes）** 菜单
2. 点击 **添加路由（Add Route）**
3. 选择模板和关联的通知渠道
4. 保存路由规则

### 5️⃣ 生成 API Key

1. 进入 **API Key（API Keys）** 菜单
2. 点击 **添加 API Key（Add API Key）**
3. 选择关联的路由
4. 保存后会生成唯一的 API Key

### 6️⃣ 调用 API 发送通知

```bash
curl -X POST "http://your-server:5400/api/service/notify" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{
    "title": "通知标题",
    "content": "通知内容",
    "push_img_url": "https://example.com/image.jpg",
    "push_link_url": "https://example.com"
  }'
```

或者在 URL 参数中传递：

```bash
curl -X POST "http://your-server:5400/api/service/notify?api_key=your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "通知标题",
    "content": "通知内容"
  }'
```

---

## 🔧 环境变量

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `TZ` | `Asia/Shanghai` | 时区配置 |
| `DATA_PATH` | `/data` | 数据存储路径（建议挂载） |
| `ADMIN_SESSION_KEY` | 自动生成 | 会话安全密钥（可选） |
| `PORT` | `5400` | 服务监听端口 |

---

## 💾 数据持久化

所有数据（数据库、配置）存储在容器内的 `/data` 目录，建议挂载到宿主机以保证数据安全。

挂载后目录结构：
```
data/
└── notifycenter.db
```

---

## � 详细功能说明

### 🎨 强大的模板系统

NotifyCenter 支持两种类型的模板，提供灵活的通知格式化能力！

#### 通用模板（General）
适用于大多数通知场景：
- ✅ 使用 Jinja2 模板语法，如 `{{title}}`、`{{content}}`
- ✅ 支持未知变量占位符：如果 JSON 中有不在模板中的字段，模板中引用的 `{{field}}` 不会报错，而是保留占位符
- ✅ 灵活自定义：可以把任意 JSON 字段添加到模板中

#### Emby 专用模板（亮点功能）
专为 Emby 媒体服务器设计，功能强大！

**主要特点：**
- 🎬 **智能事件分类**：自动将 Emby 事件映射为更细粒度的类型
  - `playback.start` → 根据媒体类型自动分为 `playback.movie`（电影播放）或 `playback.episode`（剧集播放）
  - `library.new` → 自动分为 `library.new_movie` 或 `library.new_episode`
- 🌄 **自动生成封面图片地址**：从 Emby JSON 数据生成完整的图片 URL
- ⏱️ **自动计算播放进度**：显示当前/总时长（分钟）和进度百分比
- 🕐 **时区转换**：自动把 UTC 时间转为本地时区（默认 Asia/Shanghai）
- 📦 **子模板支持**：每个事件可以配置独立的模板
- 🔍 **丰富的变量**：提供几十个 Emby 专用变量，包括：
  - `ItemName`：媒体名称
  - `SeriesName`、`Season`、`Episode`：剧集信息
  - `UserName`：用户名称
  - `DeviceName`、`Client`、`ClientIp`：设备信息
  - `CoverImgUrl`：完整封面图片地址
  - `DateLocal`：本地时间
  - 更多变量...

**模板配置说明：**
```
event_templates 格式：
{
  "playback.movie": {"title": "...", "content": "..."},
  "playback.episode": {"title": "...", "content": "..."},
  "library.new_movie": {"title": "...", "content": "..."},
  "library.new_episode": {"title": "...", "content": "..."},
  ...
}
```

---

### 🔑 API Key 详细说明

API Key 是 NotifyCenter 的入口，每个 API Key 关联：
1. **一个路由**
2. **路由关联的模板**
3. **路由关联的多个通知渠道**

**API Key 功能：**
- ✅ **独立启用/禁用**：临时禁用某个 Key 而不影响其他
- ✅ **安全验证**：支持 Header (`X-API-Key`) 或 URL 参数 (`api_key`)
- ✅ **访问统计**：可以看到最后使用时间
- ✅ **查看复制**：管理界面可以方便地复制 API Key

---

### 💡 未知 JSON 变量处理亮点

这是 NotifyCenter 最智能的特性之一！

**工作原理：**
当调用方发送未知格式的 JSON 时（比如全新的 webhook）：
1. 系统接收整个 JSON 体
2. 所有 JSON 字段都自动成为可在模板中使用的变量
3. 直接在模板中添加 `{{any_field_from_json}}` 就能使用！

**示例：**
调用方发送：
```json
{
  "custom_event": "test",
  "message": "Hello!",
  "user": "alice",
  "data": { "a": 1, "b": 2 }
}
```

模板可以写成：
```
标题：{{custom_event}}
内容：{{message}}
用户：{{user}}
其他：{{data.a}}
```

完全不需要修改代码！直接改模板就行！

---

## � 更新日志

###v0.4 (最新)
- ✅ 技术栈变更： Python (FastAPI) 改为 Go (Gin)，Jinja2 → Pongo2，SQLAlchemy → GORM，单一二进制部署替代 Python 运行时依赖。
- ✅ 部署与性能提升： 单一二进制文件、更小的 Docker 镜像、毫秒级启动、更低内存占用。
- ✅ 功能完整性： 100% 兼容原 Python 版本，包括多渠道通知、Emby 模板系统、API Key 管理、路由管理、Web 管理后台、日志系统，数据库文件可直接迁移。
- ✅ Bug 修复（11 项）： Emby 模板变量匹配、进度精度、季数/集数显示、中文截断乱码、时区崩溃、模板语法兼容、GORM 更新问题、数据库迁移安全、根路径重定向、退出登录、版本号显示。
- ✅ 模板语法说明： Pongo2 与 Jinja2 兼容，旧模板 {%% / %%} 自动修正。
- ✅ Emby 变量表： 列出所有 18 个可用变量及说明。
- ✅ 迁移指南： 从 Python 版本迁移只需停止旧容器、备份数据库、用新镜像挂载相同 data 目录即可

### v0.32
- ✅ 新增详细的功能文档，包括模板系统、API Key、未知变量处理等
- ✅ 移除不必要的 Bark 兼容接口，简化代码
- ✅ 优化项目结构，保持简洁

### v0.31
- ✅ 新增版本管理功能，所有页面显示版本号
- ✅ 新增后端日志详情查看功能
- ✅ 自动生成随机会话安全密钥
- ✅ 优化 Dockerfile，支持国内镜像源
- ✅ 更多细节优化

---

## 🎯 技术栈

- **后端**：FastAPI (Python)
- **数据库**：SQLite (SQLAlchemy ORM)
- **前端**：HTML + Jinja2 + Tailwind CSS
- **部署**：Docker

---

## 📄 许可证

本项目开源，欢迎自由使用和贡献！
