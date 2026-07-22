#  NotifyCenter - 多渠道通知中心

一个功能完整的多渠道通知转发服务，支持 Bark、Telegram、Mattermost、企业微信、PushDeer、Webhook 等多种通知方式。通过统一的 API 接口，快速将通知推送到多个渠道，支持自定义通知模板和路由规则。

---

##  v0.5 更新说明：新增多个推送渠道

v0.5 版本在 v0.4 Go 重写的基础上，重点扩展了通知渠道的覆盖范围，新增企业微信（应用消息、群机器人）和 PushDeer 三种推送方式，并对配置存储、日志系统等基础能力进行了修复与增强。

### 新增通知渠道

| 渠道 | 类型 | 说明 |
|------|------|------|
| **企业微信（应用消息）** | `wecom` | 通过企业自建应用推送到企业微信 App，需配置 CorpID / CorpSecret / AgentID，支持指定接收人/部门/标签，需在企业微信后台配置可信 IP |
| **企业微信（群机器人）** | `wecom_webhook` | 通过群机器人 Webhook 推送到企业微信群，仅需 Webhook Key，无需可信 IP，支持 text / markdown 消息类型，同时兼容填写完整 URL 和单独 key 值两种方式 |
| **PushDeer** | `pushdeer` | 开源自建友好的推送服务，支持官方服务器和自建服务器，支持 markdown / text / image 三种消息类型 |

**v0.5 支持的完整渠道列表**：Bark、Telegram、Mattermost、企业微信（应用消息）、企业微信（群机器人）、PushDeer、自定义 Webhook。

### Demo 初始化数据完善

首次启动时会自动创建全部支持渠道的示例配置（占位符值），方便用户快速了解每种渠道的必填字段：

- Demo Bark
- Demo Telegram
- Demo Mattermost
- Demo 企业微信应用（新增）
- Demo 企业微信群机器人（新增）
- Demo PushDeer（新增）
- Demo 自定义 Webhook

### Bug 修复

- **渠道配置存储损坏**：修复 `Channel.Config` 字段使用标准 `json.RawMessage` 时未实现 GORM 的 `Scan/Value` 接口，导致配置在数据库读写过程中被破坏、编辑弹窗字段为空的问题；统一改用自定义 `models.JSONRawMessage`。
- **API 返回配置格式错误**：修复 `ChannelResponse.Config` 返回为字符串导致前端 `ch.config[key]` 取值 `undefined` 的问题，改为反序列化为对象返回。
- **Bark 配置类型不匹配**：修复 `autoCopy` / `isArchive` 字段字符串与整型不一致导致反序列化失败的问题。
- **企业微信 Webhook Key 兼容**：`wecom_webhook` 渠道自动识别输入内容——填写完整 URL 时自动提取 `key` 参数，填写单独 key 值时直接使用。

### 前端改进

- 渠道管理页面新增企业微信、PushDeer 三种类型的动态配置表单，字段随类型自动切换。
- 编辑弹窗正确回显所有渠道的完整配置字段。

---

##  v0.4 更新说明：Go 语言重写

NotifyCenter 0.4 版本是一次全面的技术栈升级，将整个项目从 **Python (FastAPI)** 重写为 **Go (Gin)**，在保持全部功能、页面、接口、数据库结构和初始化数据完全兼容的前提下，带来显著的性能和部署体验提升。

### 技术栈变更

| 模块 | 旧版本 (v0.3x, Python) | v0.4 (Go) |
|------|------------------------|-----------|
| 后端框架 | FastAPI (Python) | Gin (Go) |
| 模板引擎 | Jinja2 | Pongo2（兼容 Jinja2 语法） |
| ORM | SQLAlchemy | GORM |
| 数据库 | SQLite | SQLite（完全兼容，直接迁移） |
| 会话管理 | Starlette Session | gin-contrib/sessions（Cookie-based） |
| 进程管理 | Uvicorn | Tini（容器内） |
| 部署方式 | Python + 依赖 | 单一静态二进制文件 |

### 部署与性能提升

- **单一二进制文件**：编译后仅一个可执行文件，无需安装 Python 运行时和依赖包。
- **镜像体积更小**：基于 Alpine Linux 的多阶段构建，最终镜像仅包含运行时必要组件。
- **启动速度更快**：毫秒级启动，无需等待 Python 解释器加载。
- **内存占用更低**：Go 原生并发模型，资源开销显著降低。
- **CGO 编译**：使用 SQLite 原生驱动，数据库操作性能更优。

### 功能完整性

v0.4 版本保持了与原 Python 版本 100% 的功能兼容：

- **多渠道通知**：Bark、Telegram、Mattermost、Webhook
- **Emby 模板系统**：智能事件分类、自动封面生成、播放进度计算、时区转换、子模板支持
- **API Key 管理**：生成、启用/禁用、绑定路由
- **路由管理**：灵活配置模板与渠道的关联
- **Web 管理后台**：完整的后台管理界面
- **日志系统**：通知日志和后端日志，支持详情查看
- **数据库兼容**：可直接使用原 Python 版本的 `notifycenter.db` 数据文件

### Bug 修复

此版本修复了多个从 Python 版本继承或迁移过程中发现的问题：

- **Emby 模板变量匹配**：修复 `Client` 为 null、`ItemRuntime` 计算错误的问题，正确从 `Session.Client` 和 `Item.RunTimeTicks` 提取数据。
- **进度数值精度**：修复进度百分比显示多余小数位的问题（如 `50.100000` → `50.1`）。
- **季数/集数显示**：修复浮点数显示问题（如 `S1.000000E3.000000` → `S1E3`）。
- **中文简介截断乱码**：修复按字节截取导致中文乱码的问题，改为按 Unicode 字符截取。
- **时区崩溃**：修复 Docker 容器中 `Asia/Shanghai` 时区加载失败导致 panic 的问题，增加固定偏移量兜底，并在镜像中安装 `tzdata`。
- **模板语法兼容**：自动归一化处理 `{%%` / `%%}` 等旧版模板语法，确保与 Pongo2 引擎兼容。
- **GORM 更新问题**：修复 SQLite WAL 模式下 `Save` 方法更新失效的问题，改用 `Updates` 显式更新字段。
- **数据库迁移安全**：修复自动迁移失败时误删全部表和数据的问题，改为仅记录警告。
- **根路径重定向**：访问根路径时自动跳转到 `/admin/login`（未登录）或 `/admin/dashboard`（已登录）。
- **退出登录**：修复退出功能返回 404 的问题。
- **版本号显示**：修复日志中版本号显示为 `v2.0.0` 的问题，正确读取 `VERSION` 文件显示 `v0.4`。

### 模板语法说明

使用 **Pongo2** 作为模板引擎，语法与 Jinja2 高度兼容：

- 变量输出：`{{ variable }}`
- 条件判断：`{% if condition %}...{% endif %}`
- 默认值：`{{ variable or "默认值" }}`
- 变量定义检查：`{% if variable is defined %}...{% endif %}`

> 旧模板中的 `{%%` / `%%}` 语法会被自动修正，无需手动修改已有模板。

---

##  核心功能

###  支持的通知渠道

| 渠道 | 类型标识 | 简要说明 |
|------|---------|---------|
| **Bark** | `bark` | iOS 设备即时推送，支持自建服务器 |
| **Telegram** | `telegram` | Telegram Bot 消息，支持 Markdown / HTML |
| **Mattermost** | `mattermost` | Mattermost 团队协作 Incoming Webhook |
| **企业微信（应用消息）** | `wecom` | 企业微信自建应用推送到员工 App |
| **企业微信（群机器人）** | `wecom_webhook` | 企业微信群机器人 Webhook，无需可信 IP |
| **PushDeer** | `pushdeer` | 开源推送服务，支持官方 / 自建服务器 |
| **自定义 Webhook** | `webhook` | 通用 HTTP Webhook，可自定义方法、请求头、请求体 |

#### 渠道详细说明

<details>
<summary><b>Bark</b> — iOS 即时推送</summary>

- **必填**：设备码（Device Key）
- **可选**：服务器地址、推送级别（active / timeSensitive / passive）、角标、分组、图标 URL、铃声、自动复制、保存归档
- **消息映射**：`imgURL` → `icon`；`linkURL` → `url`
- **官方文档**：[Bark 官方文档](https://bark.day.app/)
</details>

<details>
<summary><b>Telegram</b> — Bot 消息推送</summary>

- **必填**：Bot Token、Chat ID
- **可选**：解析模式（Markdown / HTML / MarkdownV2）、禁用通知、保护内容、回复消息 ID
- **获取方式**：Bot Token 由 [@BotFather](https://t.me/BotFather) 生成；Chat ID 可通过 [@userinfobot](https://t.me/userinfobot) 获取
</details>

<details>
<summary><b>Mattermost</b> — Incoming Webhook</summary>

- **必填**：服务器地址、Webhook Key
- **可选**：默认频道、用户名、头像图标 URL、头像表情
- **获取方式**：在 Mattermost 后台创建 Incoming Webhook
</details>

<details>
<summary><b>企业微信（应用消息）</b> — 推送到员工 App</summary>

- **必填**：CorpID、CorpSecret、AgentID
- **可选**：接收人（`to_user`）、接收部门（`to_party`）、接收标签（`to_tag`），均为空时默认 `@all`
- **特别注意**：需在企业微信管理后台的应用详情页配置"企业可信 IP"，否则会返回 `60020: not allow to access from your ip`
- **官方文档**：[发送应用消息](https://developer.work.weixin.qq.com/document/path/90236)
</details>

<details>
<summary><b>企业微信（群机器人）</b> — 推送到企业微信群</summary>

- **必填**：Webhook Key（支持完整 URL 或独立 key 值两种格式）
- **可选**：消息类型（`text` 纯文本 / `markdown` 富文本，默认 text）
- **优势**：无需配置可信 IP，添加群机器人即可获取 Webhook 立即使用
- **官方文档**：[群机器人配置说明](https://developer.work.weixin.qq.com/document/path/91770)
</details>

<details>
<summary><b>PushDeer</b> — 开源推送服务</summary>

- **必填**：PushKey
- **可选**：服务器地址（默认 `https://api2.pushdeer.com`，支持自建）、消息类型（`markdown` 默认 / `text` / `image`）
- **消息类型说明**：
  - `markdown`：`title` 作为标题，`content` 作为富文本正文，自动追加图片和链接
  - `text`：`title` + `content` 拼接为纯文本
  - `image`：以图片 URL（`imgURL`）作为消息内容
- **官方项目**：[PushDeer](https://github.com/easychen/pushdeer)
</details>

<details>
<summary><b>自定义 Webhook</b> — 通用 HTTP 推送</summary>

- **必填**：Webhook URL、请求方法（POST / GET）、请求体模板
- **可选**：请求头（JSON 格式）、超时时间（秒）
- **模板变量**：`{{title}}`、`{{content}}`、`{{img_url}}`、`{{link_url}}`
- **适用场景**：需要对接任意第三方接收端，或将通知转发到 Slack、Discord、飞书等未内置的服务
</details>

###  核心特性

-  **统一 API 接口**：简单 HTTP 请求即可发送通知
-  **模板系统**：支持通用模板和 Emby 专用模板
-  **路由管理**：灵活配置 API Key 与模板、渠道的关联
-  **API Key 管理**：支持生成、启用/禁用、删除 API Key
-  **完整日志**：推送日志和后端日志，支持查看详情
-  **Web 管理后台**：友好的用户界面，轻松配置所有功能
-  **安全机制**：基于 Cookie 的会话管理

---

##  快速开始

### 使用 Docker 运行（推荐）

```bash
docker run -d \
  --name notifycenter \
  -p 5400:5400 \
  -v ./data:/app/data \
  -e TZ=Asia/Shanghai \
  --restart unless-stopped \
  ttt216/notifycenter:0.5
```

### 使用 Docker Compose 运行

```yaml
version: '3.8'

services:
  notifycenter:
    image: ttt216/notifycenter:0.5
    container_name: notifycenter
    ports:
      - "5400:5400"
    volumes:
      - ./data:/app/data
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - DATA_PATH=/app/data
      - PORT=5400
```

### 从 Python 版本迁移

1. 停止旧容器。
2. 备份 `data/notifycenter.db` 文件。
3. 使用 v0.5 镜像创建新容器，挂载相同的 `data` 目录。
4. 数据库结构完全兼容，无需任何迁移操作。

---

##  访问地址

- **管理后台**：http://localhost:5400/admin/login
- **默认账号**：`admin` / `123456`

>  首次登录请立即修改密码！

---

##  使用说明

### 1. 登录系统

1. 访问 http://localhost:5400/admin/login
2. 使用默认账号登录
3. 首次登录会强制要求修改密码

### 2. 配置通知渠道

#### Bark 配置：
1. 进入 **渠道（Channels）** 菜单
2. 点击 **添加渠道** 选择 **Bark**
3. 填写设备码（Device Key），可从 Bark App 获取
4. 可选配置：服务器地址、推送级别、角标等

#### Telegram 配置：
1. 从 @BotFather 获取 Bot Token
2. 使用 @userinfobot 获取 Chat ID
3. 在系统中添加 Telegram 渠道并填写配置

#### Mattermost 配置：
1. 在 Mattermost 中创建 Incoming Webhook
2. 获取 Server URL 和 Webhook Key
3. 在系统中添加 Mattermost 渠道

#### 企业微信（应用消息）配置：
1. 在 [企业微信管理后台](https://work.weixin.qq.com) 创建自建应用
2. 获取 **CorpID**（"我的企业"页面）、**AgentID** 和 **Secret**（应用详情页）
3. 在系统中添加"企业微信（应用消息）"渠道并填写配置
4. **重要**：需要在应用详情页的"企业可信IP"中配置服务器出口 IP，否则会返回 `60020: not allow to access from your ip` 错误
5. 可通过 `to_user` / `to_party` / `to_tag` 指定接收人/部门/标签，留空默认 `@all`

#### 企业微信（群机器人）配置：
1. 在企业微信群中添加"群机器人"，获取 Webhook 地址
2. 在系统中添加"企业微信（群机器人）"渠道
3. **Webhook Key** 支持两种格式：
   - 完整 URL：`https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx`
   - 只填 key 值：`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
4. 选择消息类型：`text`（纯文本）或 `markdown`（富文本）
5. 相比应用消息，群机器人**无需配置可信 IP**，配置更简单

#### PushDeer 配置：
1. 在 PushDeer App 中生成 **PushKey**
2. 在系统中添加 PushDeer 渠道
3. **服务器地址**：默认 `https://api2.pushdeer.com`，支持自建服务器
4. **消息类型**：`markdown`（默认）/ `text` / `image`（image 类型时以图片 URL 作为消息内容）

#### Webhook 配置：
1. 添加自定义 Webhook 渠道
2. 支持自定义请求方法、请求头、请求体模板
3. 支持变量：`{{title}}`、`{{content}}`、`{{img_url}}`、`{{link_url}}`

### 3. 设置模板

1. 进入 **模板（Templates）** 菜单
2. 点击 **添加模板**
3. 配置模板标题和内容，使用 Pongo2/Jinja2 语法

### 4. 创建路由

1. 进入 **路由（Routes）** 菜单
2. 点击 **添加路由**
3. 选择模板和关联的通知渠道
4. 保存路由规则

### 5. 生成 API Key

1. 进入 **API Key** 菜单
2. 点击 **添加 API Key**
3. 选择关联的路由
4. 保存后会生成唯一的 API Key

### 6. 调用 API 发送通知

```bash
curl -X POST "http://your-server:5400/api/service/notify?api_key=your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "通知标题",
    "content": "通知内容"
  }'
```

---

##  Emby 专用模板

NotifyCenter 完整支持 Emby 模板系统，专为 Emby 媒体服务器 Webhook 设计：

**智能事件分类：**
- `playback.start` / `playback.stop` / `playback.pause` / `playback.unpause` → 自动按媒体类型映射为 `playback.movie` 或 `playback.episode`
- `library.new` → 自动映射为 `library.new_movie` 或 `library.new_episode`

**可用变量：**

| 变量 | 说明 |
|------|------|
| `Event` | 事件类型 |
| `UserName` | 用户名称 |
| `ItemName` | 媒体名称 |
| `ItemType` | 媒体类型（Movie/Episode） |
| `ItemYear` | 发行年份 |
| `ItemOverview` | 简介（自动截断至 200 字符） |
| `SeriesName` | 剧集名称 |
| `Season` | 季数（整数） |
| `Episode` | 集数（整数） |
| `DeviceName` | 设备名称 |
| `Client` | 客户端名称 |
| `ClientIp` | 客户端 IP |
| `ServerName` | 服务器名称 |
| `ServerVersion` | 服务器版本 |
| `DateLocal` | 本地时间（自动转换时区） |
| `CoverImgUrl` | 封面图片完整 URL |
| `ProgressMinutes` | 当前播放进度（分钟） |
| `TotalMinutes` | 总时长（分钟） |
| `ProgressPercent` | 播放进度百分比 |

**子模板配置格式：**

```json
{
  "playback.movie": {"title": "...", "content": "..."},
  "playback.episode": {"title": "...", "content": "..."},
  "library.new_movie": {"title": "...", "content": "..."},
  "library.new_episode": {"title": "...", "content": "..."}
}
```

---

##  环境变量

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `TZ` | `Asia/Shanghai` | 时区配置 |
| `DATA_PATH` | `./data`（容器内 `/app/data`） | 数据存储路径 |
| `ADMIN_SESSION_KEY` | `notifycenter-session-key-2026` | 会话安全密钥 |
| `PORT` | `5400` | 服务监听端口 |
| `GIN_MODE` | `release` | 运行模式（release/debug/test） |

---

##  数据持久化

所有数据存储在 `DATA_PATH` 目录下的 `notifycenter.db` 文件中，建议挂载到宿主机。

```
data/
└── notifycenter.db
```

---

##  技术栈

- **后端**：Go + Gin
- **模板引擎**：Pongo2（兼容 Jinja2 语法）
- **数据库**：SQLite（GORM）
- **前端**：HTML + Tailwind CSS + Vanilla JS
- **部署**：Docker（多阶段构建，Alpine Linux）

---

##  更新日志

### v0.5（当前版本）
-   **新增企业微信（应用消息）渠道**：通过企业自建应用推送到企业微信 App
-   **新增企业微信（群机器人）渠道**：通过 Webhook 推送到企业微信群，无需可信 IP，Key 支持完整 URL 或独立 key 值
-   **新增 PushDeer 渠道**：支持官方及自建服务器，支持 markdown / text / image 三种消息类型
-   **Demo 数据补齐**：首次启动示例数据涵盖所有支持渠道
-   **配置存储修复**：修复渠道配置字段使用标准 `json.RawMessage` 未实现 GORM `Scan/Value` 接口，导致编辑弹窗字段为空的问题
-   **配置返回格式修复**：修复渠道详情 API 返回 `config` 为字符串导致前端取值 `undefined` 的问题
-   **前端表单增强**：渠道管理页面新增企业微信、PushDeer 三种类型的动态配置表单和编辑回显

### v0.4
-   **Go 语言全面重写**：从 Python (FastAPI) 迁移至 Go (Gin)
-   **模板引擎升级**：Jinja2 → Pongo2，保持语法兼容，自动修正旧模板语法
-   **单一二进制部署**：编译为静态二进制文件，无需 Python 运行时
-   **Docker 镜像优化**：多阶段构建，基于 Alpine Linux，镜像更小
-   **时区处理增强**：Docker 镜像内置 tzdata，代码层增加固定偏移量兜底
-   **Emby 模板修复**：变量映射、进度精度、季数/集数显示、中文截断乱码等全面修复
-   **路由重定向**：根路径自动跳转至登录页或仪表盘
-   **版本号显示**：所有页面底部显示项目名称和版本号
-   **数据库兼容**：完全兼容 Python 版本的 SQLite 数据库

### v0.32（Python 版本）
-   新增详细的功能文档
-   移除不必要的 Bark 兼容接口
-   优化项目结构

### v0.31（Python 版本）
-   新增版本管理功能
-   新增后端日志详情查看
-   自动生成随机会话安全密钥
-   优化 Dockerfile

---

##  许可证

本项目基于 MIT 许可证。
