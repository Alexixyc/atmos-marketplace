# 开发、构建与 GitHub 发布

日常修改在开发仓库完成，发行仓库只接收构建产物与用户文档。

| 用途 | GitHub | SSH | 分支 |
| --- | --- | --- | --- |
| 开发源码（私有） | [weather-Atmos-codex-plugin](https://github.com/Alexixyc/weather-Atmos-codex-plugin) | `git@github.com:Alexixyc/weather-Atmos-codex-plugin.git` | `master` |
| 用户发行（公开） | [atmos-marketplace](https://github.com/Alexixyc/atmos-marketplace) | `git@github.com:Alexixyc/atmos-marketplace.git` | `main` |

以下命令从开发仓库根目录执行。需要 Node.js 22+、Git，以及拥有两个仓库写权限的 GitHub SSH 身份。用户安装说明的源文件是 `docs/marketplace-readme.md`，打包时生成发行仓库根 `README.md`。

## 1. 开始迭代

首次获取源码：

```bash
git clone git@github.com:Alexixyc/weather-Atmos-codex-plugin.git
cd weather-Atmos-codex-plugin
npm ci
```

已有工作副本时，先用 `git status` 检查本地改动，提交自己的改动后再同步：

```bash
git pull --ff-only origin master
npm ci
npm run dev
```

打开 `http://127.0.0.1:4317` 开发。React 界面在 `src/`，天气和工具在 `server/`，城市资源在 `shared/` 与 `public/landscapes/`，自然语言工作流在 `skills/weather/SKILL.md`。

## 2. 更新版本与文档

当前发行版本为 `1.1.2`（2026-10-02），历史见[版本记录](changelog.md)。后续每次正式发布提高版本，例如下次修复使用 `1.1.3`、新增兼容功能使用 `1.2.0`。以下以发布 `1.1.3` 为例，执行时替换为实际版本。

```bash
npm version 1.1.3 --no-git-tag-version
```

随后把 `plugin.json` 与 `.codex-plugin/plugin.json` 的 `version` 改为同一版本。`npm version` 会同步 `package.json` 和 `package-lock.json`；打包脚本还会检查两份插件清单的一致性。宿主客户端版本标识位于 `src/host.ts`，侧栏版本位于 `src/App.tsx`，发行时也应同步。MCP 与 Web 服务版本由 `server/loopback.ts` 从 `package.json` 读取，在构建时打包。保留插件名 `atmos-weather` 和市场名 `atmos-local`。

更新用户指南、相关技术文档及必要的截图。依赖变化后更新 `THIRD_PARTY_NOTICES.md`；更换照片后更新 `docs/landscapes.md` 及来源记录。分发需要保留上游作者、许可和改动说明。

MCP UI resource URI 由插件版本生成（当前 `ui://atmos/weather-1.1.2.html`），正式发布时以新版本重新构建，避免宿主沿用旧 HTML。当前内嵌影片单独固定到已经发布的 `v1.1.1` 标签；应用升级无需随意改变未变动的媒体地址。

更换影片的顺序：先将新文件发布到可公开读取的固定 Git 标签或提交，确认 HTTPS 地址可用、Range 请求真实返回 206、文件大小与 SHA-256 正确，再修改 `shared/timelapses.ts` 的 `embeddedSrc` 并发布引用它的应用版本。不要引用变化中的 `main` 或尚未发布的标签，不移动已发布标签；同时更新本地 `src` 的 `v` 参数、影片说明及打包文件。这样新应用在安装时就能访问有效媒体。

## 3. 验证并提交开发源码

```bash
npm test
npm run build
npm run test:mcp
npm run package:plugin
npm run test:marketplace
```

`build` 含 TypeScript 检查。`test:mcp` 会调用真实天气与地名服务，启动本机测试端口 `4318`，并更新 `docs/mcp-verification.json`。`test:marketplace` 需要支持 `app-server` 的 Codex CLI，只读检查发行目录的 MCP 与 Skill；它不替换当前插件来源。`ATMOS_CODEX_CLI` 可指定 CLI 路径。

功能或 UI 有变化时，另做实际交互验证并更新 `docs/verification.md`；插件元数据读取成功不等于原生内嵌界面已显示。

1.1.2 的媒体检查需分别覆盖两条路径：MCP Apps 的固定 HTTPS 影片与独立 Web 的包内文件。已核实的桌面宿主会过滤 HTTP loopback `resourceDomains`，不是漏打包，不能只验证本机 Range 就判定内嵌可播。核对照片回退、20 秒前台可见等待超时、重试、独立网页按钮和减少动态效果提示；原生图形播放未现场验收时，发布说明必须保留该边界。

检查本次变更，提交开发源码：

```bash
git status --short
git add .
git diff --cached --stat
git commit -m "Release Atmos 1.1.3"
git push origin master
```

开发仓库的 `.gitignore` 排除 `node_modules/`、`dist/`、`dist-server/`、`release/`、`.cache/` 及环境配置文件。推送前检查暂存内容。若远程出现新提交，用 `git fetch` 检查并整合后重试；发布流程不需要强制推送。

## 4. 从已提交源码重新打包

确认 `git status --short` 为空，再运行：

```bash
npm run build
npm run package:plugin
```

`release/release.json` 会记录源码的 Git 提交号，便于追溯。正式发行应来自干净且已推送的源码提交；无提交历史时该字段为 `null`，仅适合首次打包检查。

生成目录：

```text
release/
├── .agents/plugins/marketplace.json
├── .gitignore
├── README.md
├── THIRD_PARTY_NOTICES.md
├── release.json
└── atmos-weather/
    ├── plugin.json / mcp.json
    ├── .codex-plugin/ / .mcp.json
    ├── skills/ / assets/
    ├── dist/ / dist-server/mcp.cjs
    ├── docs/
    └── THIRD_PARTY_NOTICES.md
```

市场索引的 `source.path` 固定为 `./atmos-weather`，相对于发行仓库根目录。完整前端和服务端已构建，用户无需 `npm install`。`package:plugin` 不自动创建 ZIP，也不推送 Git；历史 ZIP 不参与以下同步流程。

## 5. 同步并推送发行仓库

首次准备维护用的发行副本：

```bash
git clone --branch main git@github.com:Alexixyc/atmos-marketplace.git .cache/atmos-marketplace
```

后续发布复用该目录。先确认其工作区干净，检查 origin 指向正确的发行仓库，然后同步远程：

```bash
git -C .cache/atmos-marketplace status --short
git -C .cache/atmos-marketplace remote -v
git -C .cache/atmos-marketplace pull --ff-only origin main
```

同步本插件目录及指定发行文件，保留发行仓库的 Git 历史和其他内容：

```bash
mkdir -p .cache/atmos-marketplace/atmos-weather
mkdir -p .cache/atmos-marketplace/.agents/plugins
rsync -a --delete release/atmos-weather/ .cache/atmos-marketplace/atmos-weather/
cp release/.agents/plugins/marketplace.json .cache/atmos-marketplace/.agents/plugins/marketplace.json
cp release/README.md release/.gitignore release/release.json release/THIRD_PARTY_NOTICES.md .cache/atmos-marketplace/
npm run test:marketplace -- .cache/atmos-marketplace
```

这里的删除同步只作用于 `atmos-weather/` 子目录。未来若市场包含多个插件，需要合并市场索引，保留其他插件条目。Windows 用户可使用 WSL/Git Bash 中的相应工具，或以文件复制工具完成同样的目录同步。

审阅后推送：

```bash
git -C .cache/atmos-marketplace add .agents/plugins/marketplace.json atmos-weather README.md .gitignore release.json THIRD_PARTY_NOTICES.md
git -C .cache/atmos-marketplace diff --cached --stat
git -C .cache/atmos-marketplace commit -m "Release Atmos 1.1.3"
git -C .cache/atmos-marketplace push origin HEAD:main
```

需要归档版本时，在源码提交和对应发行提交分别创建同名标签；已经发布的标签保持不变：

```bash
git tag -a v1.1.3 -m "Atmos 1.1.3 source"
git push origin v1.1.3
git -C .cache/atmos-marketplace tag -a v1.1.3 -m "Atmos 1.1.3 distribution"
git -C .cache/atmos-marketplace push origin v1.1.3
```

发布后核对两边 Git 状态与远程提交号，并从发行仓库重新拉取检查。可对回拉包运行 `node scripts/smoke-mcp.mjs /path/to/checkout/atmos-weather`（会更新本地验证记录），以及 `npm run test:marketplace -- /path/to/checkout`。如果希望保持源码工作区干净，可在临时目录执行冒烟脚本，让验证报告写入临时目录的 `docs/`。

## 6. 用户安装、更新与回退

把 [发行仓库 README](https://github.com/Alexixyc/atmos-marketplace#readme) 发给用户。安装：

```bash
codex plugin marketplace add Alexixyc/atmos-marketplace --ref main
codex plugin add atmos-weather@atmos-local
```

更新：

```bash
codex plugin marketplace upgrade atmos-local
codex plugin add atmos-weather@atmos-local
```

然后重新打开聊天并选择插件，说“打开天气”“看看杭州天气”或“切换到上海”。已有本地开发市场的维护者按[集成说明](integration.md)明确切换来源。

遇到需要回退的发行问题，优先在开发仓库恢复兼容行为，发布新的修复版本，保留 Git 历史。用户也可移除市场来源后用 `--ref v1.0.0` 添加已发布标签，再安装插件；固定标签后不会跟随 `main` 更新。

官方依据：[Git 市场、打包结构与更新命令](https://developers.openai.com/plugins/build/plugins)。这套流程面向 GitHub 自建市场；官方公开目录的审核与云端部署属于另一套发布流程。
