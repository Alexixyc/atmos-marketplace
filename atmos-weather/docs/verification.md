# 实际验收记录

验收日期：2026-10-01（Asia/Shanghai）。运行时天气为真实 Open-Meteo 请求，没有演示天气或硬编码天气值。

## 已验证的作品

- `npm run build`：TypeScript、Vite 单文件前端、Web 服务与独立 MCP CJS 包构建成功。
- `npm test`：41 个用例通过，覆盖时区及 DST、WMO 缺失值、天气归一化、缓存/并发合并/刷新节流、错误回退、坐标身份隔离、反向解析、浏览器偏好，以及 MCP 初始空结果与深链竞态、连续深链仅最后一次生效。
- 桌面 1440×1000、1280×900；手机 390×844；窄侧栏 320×900。全部无横向溢出，小时预报有独立横向滚动区域，7 日及详情可正常浏览。
- 十城各自的天气、中文名、背景标识和独立高清图片逐一匹配；每座城市都测到 `camera-drift` 的实际变换值变化。详见 [逐城浏览器证据](ui-city-verification.json)。
- 中英文搜索：南京 / Nanjing 返回相同 GeoNames ID，结果展示江苏或云南等地区以消歧；浏览器实际选择江苏南京后获取真实天气、显示「临时背景」，收藏保留 ID、坐标、时区。
- 生产 `npm start` 在独立 4318 端口启动成功；健康检查、打包 HTML 与景观 JPEG 均返回 HTTP 200。
- 沉浸观景模式可进入/退出，小时预报左右滚动经过实际点击验证。
- 摄氏 23°C 切换为 74°F；主视觉、小时/日预报、收藏同步，切回摄氏成功。
- 手机收藏添加与删除经过实际点击验证；在城市浮层内也可管理收藏。
- 设置关闭动态风景后运行中动画为 0，静态主图仍完整可见；系统 `prefers-reduced-motion: reduce` 下照片动画为 `none`。
- 连续切换上海 → 杭州 → 东京，并对三条**真实**响应人为增加 700/350/20 ms 延迟，最终城市、天气、背景均保持东京，三请求返回后仍一致。
- 选城完成回到页面顶部；城市背景提前加载，850 ms 淡入，无白屏。
- 当地时间检查：亚洲白天、伦敦晨光、纽约夜间，巴黎晨光到日间边界符合本地日出。
- 时间轴「现在」使用当前 15 分钟模型估计，其余时段使用逐小时预报，避免当前值与整点预测混淆。
- 数据来源、模型估计、数据时间、刷新时间、缓存与过期结果分别标明。

## 定位和异常流程

[异常验收记录](ui-failure-verification.json) 将定位成功、拒绝、超时、迟到回调、天气网络失败、缓存回退、重试和图片失败逐项记录。

浏览器定位回调使用**应用级测试替身与公开城市坐标**，后续地名解析和天气查询使用真实服务。测试没有读取用户实际设备位置，也没有自动批准操作系统定位权限。真实设备权限仍由用户首次点击「使用当前位置」后决定。

Nominatim 在本机网络不稳定；已加入实际可用的 Photon / OpenStreetMap 反向解析回退。悉尼名称、实际经纬度、Australia/Sydney 时区及天气真实验证通过。中文索引缺少部分城市别名时可用英文名搜索。

## Codex 集成证据

- 宿主：macOS ChatGPT/Codex desktop 26.928.21956（12404），内置 Codex CLI 0.159.2。PATH 的旧 CLI 0.135.0 与现有 `ultra` 配置不兼容，安装脚本使用内置版本，无需修改用户配置。
- SDK：`@modelcontextprotocol/sdk` 1.31.0、`@modelcontextprotocol/ext-apps` 2.0.3；依赖锁定于 package-lock.json。
- 本地市场插件 `atmos-weather@atmos-local` 已安装并启用。
- 通过官方 app-server 的 `mcpServerStatus/list` 实际发现全部 5 个工具，`toolsError` 为 null；通过 `mcpServer/resource/read` 读取含照片的完整 MCP Apps HTML。
- 源码与安装缓存中的 MCP 服务均通过协议测试：工具发现、真实天气查询、资源读取、独立 Web 入口。
- `open_in_codex` 打开本地 Web 预览，已通过 **Codex In-app Browser** 实际读取上海天气、预报和来源说明并查看界面截图。
- **原生 MCP Apps 渲染尚未声称验收通过**：当前聊天的工具目录在安装后未热刷新；原生 Codex 窗口不允许自动化访问。官方界面资源及工具已准备并由宿主成功加载，需刷新插件或在新聊天中选择观晴后调用入口，确认原生侧栏实际渲染。当前可使用已验证的应用内浏览器入口。

详见 [宿主证据](codex-host-verification.json) 与 [安装集成说明](integration.md)。

## 实际运行截图

- [杭州桌面](screenshots/hangzhou-desktop.png)
- [上海桌面](screenshots/shanghai-desktop.png)
- [东京桌面](screenshots/tokyo-desktop.png)
- [巴黎手机](screenshots/paris-mobile.png)
- [东京 320px 侧栏](screenshots/tokyo-sidebar.png)
- [手机城市选择器](screenshots/city-picker-mobile.png)

照片是授权静态作品配合动态效果，并非现场摄像头；拍摄时段可能与当前时间不同，界面按实时预报及城市日出日落施加色调。授权详情完整保存在 [素材清单](landscapes.md)。
