# 尚友

> 颂其诗，读其书，论其世。——《孟子·万章下》

古诗文分级阅读应用（Web + Android），支持 **PWA 离线阅读**。

## 内容规模

| 门类 | 篇数 | 内容特点 |
|------|------|----------|
| 蒙学 | 174 | 三百千、声律启蒙、幼学琼林等蒙学经典 |
| 初级 | 845 | 唐诗三百、千家诗、古文观止、说文部首 |
| 进阶 | 1471 | 诗经楚辞、经史子集、文选文心，全本与选篇 |

**合计 2491 篇 / 52 部**，每篇配白话段译、逐字训诂、难句解析、名物文化与句式讲评。

## 功能

- **读书次第**：八阶循序（识字蒙韵 → 属对声律 → 小学训诂 → 孝经四书 → 五经 → 史鉴 → 诸子 → 辞章），首页「今日一篇」按次第推进
- **五层详注**：训诂 / 难句 / 名物 / 句式 / 诗艺，诗词与名篇另附典故
- 原文 + 白话段译 + 注释；断句练习、背诵模式、生词与断句错题间隔复习
- **划线笔记**：选中原文即浮出工具条，四色批注（朱/黛/青/黄）＋按语；阅读页底部列本篇划线，`/notes` 汇总全库、可回原文定位
- 朗读（多音字矫正）、每日提醒（原生端）
- **科举应试**：县试 → 府试 → 院试 → 乡试 → 会试 → 殿试，考过授童生 / 秀才 / 举人 / 贡士 / 进士，殿试分三甲
- 按篇名、作者搜索；作者小传；收藏；学习数据备份（v5，含科举成绩与划线笔记）
- PWA 离线缓存

## 开发

```bash
npm install
npm run dev        # 前端 5173 + 服务端 3001
```

## 数据维护

早期生成脚本在 `scripts/levels/*.mjs`（`npm run build:data`）。日常维护的**事实源是 `尚友客户端/src/data/json/*.json` 的顶层字段**；
`books[].articles` 是嵌入副本，改完跑 `npm run gate` 里的 `sync-embed` / `sync-db` 重建，不要手改副本。
改动 `linguistics` 后必须先跑 `export-ling-slices`，Web 版由 `prune-ling-for-web` 剔除分片。

## 构建

```bash
npm run build
npm run preview
```

### 双击入口（Windows）

两个分类目录共 3 个 bat，双击即可，不必记命令：

| 目录 | 脚本 | 作用 |
|---|---|---|
| `胜迹图服务端/` | `启动服务.bat` | 默认重启 3001 全站并打开管理页与首页；`dev` 参数改跑 Vite 5173 + 接口 3001，`wall` 参数只跑 3002 题壁服务 |
| `尚友客户端/` | `打包APK.bat` | 重建 Web 资源 → 同步 Capacitor → 编译自用签名 APK 到 `尚友客户端/release/guwen-release-devkey.apk` |
| `尚友客户端/` | `项目体检.bat` | 内容质检 / 排版体检 / 发版检查，支持 `quality` `readability` `release` `build` `check` 参数 |

原先 6 个脚本已合并为 3 个：`体检.bat` 只是 `audit.bat` 的转发别名，能力并入 `项目体检.bat`；旧 `启动服务.bat`（dev 模式）与 `serve-community.bat`（3002 独立服务）并入 `启动服务.bat` 的 `dev` / `wall` 参数。旧文件没有永久删除，归档在 `.workbuddy/archive/2026-11-10-bat/`，需要时可原样还原。

**专属文件是真实实体，已物理搬进各自目录**：`尚友客户端/` 放 `src/`、`public/`、`android/`、`electron/`、`release/`、`index.html` 与 Vite / Capacitor / 两份 TS 配置（10 项）；`胜迹图服务端/` 放 `server/`（1 项）。**共用条目仍留在项目根**——`package.json`、`node_modules/`、`data/`、`dist/`、`scripts/`、`tests/`、`README.md` 等 13 项，在两个目录各有一份 NTFS 目录联接（junction，目录）或硬链接（文件）作为视图，打开编辑就是改根下的真实文件，删链接不影响实体。`尚友客户端/` 共 26 项（13 实体 + 13 链接），`胜迹图服务端/` 共 16 项（3 实体 + 13 链接）。`tsconfig.json` 的项目引用是相对根解析的，只留在根、不建视图；`.gitignore` / `.workbuddy/` / `.cap-tmp/` 同理。归属清单在 `.workbuddy/category-links.json`。

物理拆分后工具链锚点已一并改好：三个服务模块用 `../..` 上溯到项目根；`scripts/*.mjs` 里约 230 处相对路径统一加上 `尚友客户端/` 或 `胜迹图服务端/` 前缀；`vite.config.ts` 以 `尚友客户端/` 为 root、`../dist` 为输出目录；`capacitor.config.ts` 的 `webDir` 改为 `../dist`，`cap sync` 在客户端目录下执行；`package.json` 的 dev / preview / build / electron 命令全部指向新位置。`npm test` 仍由 `scripts/run-tests.mjs` 的 ASCII 目录联接绕过 vitest 中文路径问题。重建／撤销共享视图用 `.workbuddy/rebuild-category-links.ps1`（`-Check` 只统计、`-Remove` 只删链接，遇真实文件跳过）。

bat 用 `%~dp0..` 回到项目根执行，内容保持纯 ASCII（UTF-8 中文配 `chcp 65001` 会让 cmd 错读行偏移把命令粘在一起）。三个脚本都支持 `check` 只做环境自检，设置 `NO_PAUSE=1` 可跳过结束时的暂停。打完 APK 后仍需按上表把 `dist` 恢复成 Web 版。

### 产物形态与首载体积

同一份源码按目标端产出两种 `dist`，不要混用：

| 目标 | 命令 | 数据 |
|---|---|---|
| Web / PWA | `npm run clean:dist && npm run build` | 剔除 `dist/data/guwen.db`（约 8MB）与 19.5MB 详注分片，由服务端按需读根目录 `data/` |
| Android 离线 | `npm run pack:apk:release` | `CAP_ANDROID=1` 构建，保留本地库与 2491 个详注分片，断网可读 |

首载只带入口 chunk 与插图；sql-wasm、典故例句等按路由动态 import。简繁字典 `cn2t` 同样是动态 import，但**必须进 Service Worker 预缓存**：离线时首次切换繁体若没有缓存会静默失效。其余大 chunk 与 2491 个预渲染页用 `globIgnores` 排除，避免首次安装下载整站。

发版顺序：`npm run gate` → `pack:apk:release` → `npm run clean:dist && npm run build` 把 dist 恢复成 Web 版 → `npm run audit:release`。打完 APK 不恢复 dist，Web 版会多带近 28MB 离线数据。

## 版权与上架物料

`尚友客户端/release/copyright/` 存放软件著作权登记与 APP 电子版权认证的申请材料（两份申请表填报稿、软件说明书 PDF、源程序 60 页鉴别材料、独立开发声明、办理指引）。改版本号或软件全称后必须跑 `npm run copyright:materials` 重新生成：页眉、源程序抽样与填报稿里的源程序量会自动同步。主体、待填字段与提交步骤见 `尚友客户端/release/store/上架清单.md`。

## 巡检（只读）

三个体检脚本，都不改数据，随时可跑：

```bash
npm run audit              # 全部跑一遍
npm run audit:quality      # 内容质检：结构 / 段译 / 正文 / 五层 / 重复
npm run audit:readability  # 排版体检：长块 / 缺断句 / 文体误判 / 段落缺译
npm run audit:release      # 上架检查：产物完整性 / 图标 / sitemap / 体积 / 合规字段
```

Windows 下也可以直接双击 `尚友客户端/项目体检.bat`，支持
`quality` / `readability` / `release` / `build` / `check` 参数。
报告写到 `.workbuddy/reports/`。有阻断性问题时退出码为 1，可直接接 CI。

## 技术栈

- React 19 + TypeScript + Vite 6
- React Router 7 + vite-plugin-pwa
