# Codex 插件集成与验证

观晴同时提供独立 Web 界面与 MCP Apps。天气查询无需付费 API Key；插件为本地个人用途设计，无需自建云端服务或开发者账号。1.1.2 的内嵌纽约影片需要访问 GitHub HTTPS 媒体地址；独立 Web 界面使用包内影片。

## 从 GitHub 安装发行版

发行仓库为 [Alexixyc/atmos-marketplace](https://github.com/Alexixyc/atmos-marketplace)，跟踪 `main`。需要 Git、Node.js 22+，以及支持 `plugin` 子命令的 Codex CLI。

```sh
codex plugin marketplace add Alexixyc/atmos-marketplace --ref main
codex plugin add atmos-weather@atmos-local
```

更新到 1.1.2 时运行 `codex plugin marketplace upgrade atmos-local`，再运行 `codex plugin add atmos-weather@atmos-local`，刷新 Codex 后打开新聊天。发行版内含完整构建产物，由 Codex 自动启动本机服务；无需下载开发源码、运行 npm 或配置天气 API Key。可在侧栏核对 `v 1.1.2`，再说“看看纽约天气，打开天气界面”。

## 从开发仓库安装

在项目根目录运行（Node.js 22+）：

```sh
npm ci
npm run build
npm run package:plugin
node scripts/install-plugin.mjs
```

安装脚本优先使用当前 ChatGPT 桌面应用自带的 Codex CLI，其他平台使用 `PATH` 中的 `codex`。可设置 `ATMOS_CODEX_CLI` 指定命令。脚本调用官方 `plugin marketplace add` 和 `plugin add`，不会手工覆盖用户配置。

打开新 Codex 聊天，在插件目录选择「观晴 · Atmos」，说“打开天气”“看看杭州天气”或“切换到上海”。如当前聊天尚未发现新工具，先使用独立界面；工具目录可能需要新聊天或重新加载桌面应用。

更新代码后重新运行 build、package:plugin 和安装脚本。安装源在 `release/atmos-weather`，缓存由宿主管理；不要手工修改缓存。GitHub 自建市场的发布流程见[发布指南](releasing.md)。官方公开目录需要单独的远程服务和发布审查，本项目未上架该目录。

## 开发版与发行版切换

两种来源使用相同市场标识 `atmos-local`。先运行 `codex plugin marketplace list` 查看当前来源。要从本地构建切换到 GitHub 发行版，运行：

```sh
codex plugin marketplace remove atmos-local
codex plugin marketplace add Alexixyc/atmos-marketplace --ref main
codex plugin add atmos-weather@atmos-local
```

要切回开发版，同样移除市场来源后，在开发目录运行 `node scripts/install-plugin.mjs`。该操作明确切换插件来源；运行中的聊天需重新加载。收藏和设置属于浏览器/宿主存储，不同界面之间不会自动同步。

只检查发行包而保持当前安装来源时，可运行 `npm run test:marketplace -- .cache/atmos-marketplace`。检查会调用宿主的 `plugin/list` 与指定路径的 `plugin/read`；同名市场可能在列表中去重，脚本会单独核对实际读取的插件目录。它不调用安装或配置写入接口。

## 图形入口与兼容策略

- `open_weather_app` 是唯一渲染工具；1.1.2 关联 `ui://atmos/weather-1.1.2.html`。URI 从插件版本生成，升级时隔离旧 HTML 缓存。
- HTML MIME 为 `text/html;profile=mcp-app`，使用官方 MCP Apps `App` 初始化、接收首个工具结果和调用数据工具。
- UI resource 的 `openai/ui` 元数据声明 `availableDisplayModes: ["fullscreen"]`、`preferredDisplayMode: "fullscreen"`。初始化后，只在宿主允许 fullscreen 时请求一次；宿主仍决定实际位置。
- opener 同时声明 global、thread Extensions 入口；线程标签名称为「城市天气之窗」。这些是可选能力，基本天气工具不依赖 Extensions。
- 首个工具结果传入完整城市与天气，界面直接使用，避免二次调用 opener。
- 单文件 HTML 内联本地风景照片，无外部照片域名依赖。天气与地理查询通过 MCP bridge；视频独立加载，不将约 63 MB 媒体内嵌到 HTML。
- 1.1.2 的内嵌影片从 `https://raw.githubusercontent.com` 读取，CSP 的 `resourceDomains` 仅声明该 HTTPS origin；地址固定到已发布的 `v1.1.1` 标签，不跟随 `main`。独立 Web 界面继续从插件本地服务读取包内同一影片。内容 SHA-256、完整 URL 与素材说明见[纽约影片说明](manhattan-video.md)。
- 影片首次联网需预留约 63 MB 流量，实际取决于 Range 与缓存。首次加载累计前台可见等待 20 秒后仍未就绪，界面显示失败状态并保留照片与天气，提供重试及可复制到浏览器的本地播放链接。减少动态效果、加载中和加载失败使用不同状态。
- `open_weather_app` 还返回含城市参数的 `webUrl`。它优先使用 `127.0.0.1:4317`，只复用插件版本和影片内容版本匹配的服务；旧服务或其他程序占用端口时，在系统分配的空闲 loopback 端口启动当前版本。独立 Web 链接使用实际 origin；内嵌影片的 HTTPS 地址与它分开。未支持 MCP Apps 的宿主可通过浏览器打开同一套界面。
- 浏览器定位由用户点击触发。MCP iframe 的 geolocation 权限仍由宿主决定；拒绝或宿主不支持时，界面支持手动选城。

## 工具

| 工具 | 输入 | 返回 |
|---|---|---|
| `search_cities` | `query`，中英文 | `cities`，稳定 ID、经纬度、时区、国家和地区 |
| `get_weather` | `cityId`、`city` 对象/名称或 `latitude`+`longitude`，可选 `force` | `weather` 当前模型估计、24 小时、7 天、缓存和来源信息 |
| `get_city_landscape` | 同上 | 城市、地标、图像、横竖裁切焦点、作者、来源及许可 |
| `reverse_geocode` | 用户授权的经纬度 | `city`，定位界面使用 |
| `open_weather_app` | 可选城市或经纬度；空对象有效 | 启动城市和天气、精选城市、独立界面 URL 与 UI resource |

返回同时包含结构化结果和中文摘要。只有读操作；无工具会自动请求用户定位、修改系统设置或发送聊天消息。

## 已核实的环境

2026-10-01 核实：macOS，ChatGPT 桌面版 `26.928.21956`（build `12404`），其 bundled Codex CLI `0.159.2`。系统 PATH 的旧 CLI 是 `0.135.0`，不能解析此用户配置中的 `ultra` reasoning；安装脚本因此优先使用 bundled CLI，不修改该配置。

2026-10-02 的 1.1.2 修复确认：上述官方桌面宿主会过滤 MCP Apps `resourceDomains` 中的 HTTP loopback origin，并阻止对应媒体请求。1.1.1 发行包实际上包含完整的 62,676,514 字节影片，问题并非漏打包。旧版为本地媒体添加 CSP origin 的方案未能解决该宿主限制，本版改用固定版本 HTTPS 资源。真实 HTTPS Range 请求已返回 206，独立 Web 界面正常；这些结果不等于原生 MCP Apps 已完成图形播放验收。

已安装并检查类型：`@modelcontextprotocol/sdk 1.31.0`、`@modelcontextprotocol/ext-apps 2.0.3`，精确依赖由 `package-lock.json` 固定。服务端使用 SDK 的标准资源和工具注册及官方 MCP Apps 元数据；浏览器使用 MCP Apps 2.x bridge。`npm run test:mcp` 会真实启动已构建的 stdio 服务，验证工具发现、中文搜索、真实天气、地标对应、含城市的 Web 入口、MCP HTML 资源及非法坐标拒绝，结果写入 `docs/mcp-verification.json`。协议验证不等于桌面嵌入界面已经显示；实际宿主渲染证据在交付说明中单独列明。

1.1.2 已通过官方 CLI 安装为 `atmos-weather@atmos-local`。本次通过 bundled CLI 的正式 `app-server` 协议验证：Codex 实际发现 `atmos` 服务的 5 个工具，无工具加载错误；发现并读取 `ui://atmos/weather-1.1.2.html`，资源大小为 7,649,848 字节（约 7.65 MB）；同时发现安装在 1.1.2 缓存目录中的 Skill `atmos-weather:weather`。证据见 `docs/codex-host-verification.json`。该检查没有创建新聊天，记录中的 `nativeMcpAppsRendered` 仍为 `false`，不代表原生视频已经完成视觉验收。

MCP Apps 原生内嵌视觉验收尚未完成。既有验收时，Computer Use 无法读取 ChatGPT/Codex 原生窗口，MCP Apps 专用标签目录为空；分发官方格式深链没有被当作显示成功。Codex In-app Browser 的实际 DOM 与截图已验证独立 Web 入口，包括真实天气和风景。升级到 1.1.2 后仍需在新聊天选择插件，现场确认原生内嵌影片的加载与播放；不能用 Web 播放或协议成功替代这一验收。

## 官方依据

- [插件打包、兼容清单与本地 marketplace](https://developers.openai.com/plugins/build/plugins)
- [MCP UI 与工具结果通信](https://developers.openai.com/plugins/build/chatgpt-ui)
- [OpenAI MCP Extensions：入口、深链与显示模式](https://github.com/openai/mcp-extensions/blob/main/docs/spec.md)
- [官方 MCP Apps SDK](https://github.com/modelcontextprotocol/ext-apps)

实现以已安装 SDK 类型和实际协议测试为准，不将官方预期支持表当作本机实测。
