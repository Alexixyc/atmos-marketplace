# 观晴 · Atmos 城市景观资源

核实日期：2026-10-01。十张照片均来自 Wikimedia Commons，各文件页的作者、许可及地标说明已逐项核实。原始 API 来源响应保存在 `landscape-sources.json`。这些是精选城市壁纸，**不是实时摄像头画面，也不代表当前天空实况**。

资源在本地 `public/landscapes/` 自托管；主图最长边 1800–1920 px（深圳原图 1800 px；巴黎去除原图外框后为 1902 px），渐进式 JPEG，另有 480 px 缩略图。仅进行了缩小、巴黎照片外框裁切及 JPEG 压缩，没有生成、替换或改画城市地标。界面根据横竖屏焦点裁切、叠加时间与天气色调并进行缓慢镜头运动。CC BY-SA 图片及其裁切/调色衍生图继续采用该图片对应的相同许可。照片版权与应用代码许可分开。

| 城市 | 地标 | 摄影作者 | 图片许可 | 原始来源 |
| --- | --- | --- | --- | --- |
| 北京 (`beijing`) | 天坛 · 祈年殿 | 黄坚基 | [CC BY-SA 3.0 CN](https://creativecommons.org/licenses/by-sa/3.0/cn/) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Temple_of_Heaven_by_%E9%BB%84%E5%9D%9A%E5%9F%BA.jpg) |
| 上海 (`shanghai`) | 外滩 · 陆家嘴 | Stefan Fussan | [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Shanghai_-_Skyline_Sunset_0057.jpg) |
| 杭州 (`hangzhou`) | 西湖 · 雷峰夕照 | Yinweichen | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Sunset_with_Leifeng_Pagoda_on_West_Lake.jpg) |
| 成都 (`chengdu`) | 锦江 · 安顺廊桥 | Limesave | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Anshun_Bridge_Night.jpg) |
| 深圳 (`shenzhen`) | 深圳湾 · 人才公园 | Windmemories | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:20201112_The_skyline_at_Shenzhen_Bay.jpg) |
| 香港 (`hongkong`) | 太平山 · 维多利亚港 | Romain Pontida | [CC BY-SA 2.0](https://creativecommons.org/licenses/by-sa/2.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Skyline_and_Victoria_Harbour_at_dusk,_view_from_Victoria_Peak,_Hong_Kong,_China_-_%E9%A6%99%E6%B8%AF%EF%BC%8C%E4%B8%AD%E5%9B%BD_(16215094838).jpg) |
| 东京 (`tokyo`) | 东京塔 · 城市灯火 | David Kernan | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Tokyo_Tower,_Minato_City.jpg) |
| 巴黎 (`paris`) | 塞纳河 · 埃菲尔铁塔 | Mustang Joe | [CC0](https://creativecommons.org/publicdomain/zero/1.0/deed.en) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Eiffel_Tower_sunset,_Paris_(9249818803).jpg) |
| 伦敦 (`london`) | 泰晤士河 · 塔桥 | Diliff | [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Tower_Bridge_London_Dusk_Feb_2006.jpg) |
| 纽约 (`newyork`) | 东河 · 曼哈顿天际线 | Giuseppe Milo | [CC BY 2.0](https://creativecommons.org/licenses/by/2.0) | [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Manhattan_Skyline_at_sunset_-_New_York.jpg) |

## 横屏与竖屏构图

`shared/cities.ts` 的 `focalPoint` 与 `mobileFocalPoint` 为百分比 CSS `object-position`。杭州另设 `desktopScale: 1.8`：桌面背景以底部为锚点放大，使雷峰塔处于主视觉区而不是预报卡片后方；手机保留原图比例。巴黎手机焦点调整为 `16% 46%`，使埃菲尔铁塔与左侧温度区域错开。主图保留完整构图，手机裁切聚焦当地地标：北京祈年殿、上海陆家嘴塔群、杭州雷峰塔、成都廊桥、深圳春笋大厦、香港中环与维港、东京塔、巴黎铁塔、伦敦塔桥、纽约曼哈顿。`landscape-contact-sheet.jpg` 是十城视觉检查拼图。

## 扩展规则

1. 先核实城市及地标，保存独立资源，不把别的城市照片用作替代。
2. 记录稳定城市 ID、地标、作者、原始页面、许可链接与两个屏幕方向焦点。
3. 图片下载后检查可解码、完整性和横竖构图，并保留署名。
4. 尚无精准景观的城市，天气查询正常，使用明确标注的临时渐变背景。
