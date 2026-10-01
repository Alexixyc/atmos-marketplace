# 品牌标志与开发者名称

当前开发者展示名：**Cyber Alexi**。

## 哪些字段控制展示

| 文件 | 开发者名称 | 主 logo | 输入框图标 |
| --- | --- | --- | --- |
| `plugin.json` | `author.name` 与 `extensions.com.openai.interface.developerName` | `extensions.com.openai.interface.logo` | `extensions.com.openai.interface.composerIcon` |
| `.codex-plugin/plugin.json` | `author.name` 与 `interface.developerName` | `interface.logo` | `interface.composerIcon` |

两份清单保持一致。作者字段保留包归属，`developerName` 控制 Codex 展示的开发者名称；插件自身的名称仍为「观晴 · Atmos」，稳定标识仍为 `atmos-weather`。

## 设计与资产

- [主 logo](../assets/atmos-logo.png)：1254 × 1254 透明 PNG，约 1.5 MB。深青色圆角底、玻璃拱窗、暖金太阳与水面，表达「透过一扇窗，看见城市与天气」。用于插件详情和「关于观晴」。
- [小尺寸图标](../assets/icon.svg)：128 × 128 SVG，以清晰的拱窗、太阳与一条水面线简化玻璃标志，用于输入框和应用左上角。
- [浏览器图标](../public/favicon.svg)：相同构图的简化实色 SVG，便于在标签栏识别。
- 页面开发者说明位于 `src/App.tsx` 的「关于观晴」；应用标志入口为 `src/components.tsx` 中的 `Brand()`。

主 logo 由内置图像生成工具创建，小尺寸与 favicon 使用项目原生 SVG。未更换摄影或视频资源，原有第三方署名保持不变。PNG 保留透明通道；生成原图留存，项目消费本仓库内的副本。

## 以后如何修改并让用户看到

1. 修改上表字段及 `assets/` 中资源；页面标志或 favicon 变化时同步相应入口。
2. 递增版本并同步两份清单与界面版本，执行 `npm run build`、`npm run package:plugin`、`npm run test:marketplace`。
3. 按[发布指南](releasing.md)提交源码，从干净源码重新打包，再推送发行仓库。不要直接编辑 Codex 管理的插件缓存。
4. 用户更新市场并重新安装插件，刷新 Codex、在新聊天中选择插件；运行中的旧聊天不会自动刷新展示信息。

```bash
codex plugin marketplace upgrade atmos-local
codex plugin add atmos-weather@atmos-local
```

## 1.1.1 验证 · 2026-10-02

- Codex CLI 实际 `plugin/read` 返回 `developerName: Cyber Alexi`，主图和输入框图标均解析到当前发行包的正确素材；两份清单一致。
- 桌面与 390px 手机宽度实际打开「关于观晴」，PNG 成功解码，名称显示正确，无横向溢出；页面左上角使用简化 SVG。
- 114 项测试、TypeScript、生产构建、真实天气 MCP 和媒体 Range 检查通过。图形检查针对本地 Web 界面，未将元数据验证等同于 Codex 插件目录页面的视觉验收。
- [桌面实际截图](screenshots/branding-about-desktop.png) · [手机实际截图](screenshots/branding-about-mobile.png)

## 图像生成提示词

模式：内置 image_gen，透明背景；原始输出为 1254 × 1254 PNG。

```text
Use case: logo-brand.
Asset type: production app icon and plugin logo for an existing immersive city weather app called 观晴 · Atmos.
Primary request: design one original refined app icon. A window opening onto changing daylight, quiet and elegant, with an exceptionally clear silhouette at 32 pixels.
Composition: one centered rounded-square icon, straight-on orthographic view, occupying almost the whole square canvas (about 94%), transparent corners outside the tile. Within the tile one substantial arched window frame made of luminous frosted pale glass, a single warm sunrise-gold sun inside the arch above one calm gently curved horizon/water line. Balanced generous breathing room inside the icon. The window is the unmistakable main symbol. No internal window grids, no buildings, no cloud clip-art.
Materials and mood: understated premium glass, softly polished curved edges, restrained highlights and subtle depth, cinematic quiet daylight. Sophisticated crafted app icon, not a photo of an icon. Crisp controlled edges, broad simple shapes. Preserve the existing Atmos deep teal, warm ivory and muted sunrise-gold brand palette. Dark teal enamel-like rounded-square field, ivory glass arch, warm gold sun. Gentle natural tonal variation, never saturated neon. Readable small symbol; no thin hairline strokes.
Text: none. No letters, words, developer text, numbers or watermark.
Constraints: exactly one finished icon, no mockup, no phone, no presentation board, no alternative designs, no surrounding objects. Square 1024 x 1024 output with genuinely transparent exterior corners. Tile front face must be centered, not perspective or isometric. Do not imitate an existing weather app logo.
```
