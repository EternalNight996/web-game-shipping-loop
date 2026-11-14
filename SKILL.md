---
name: web-game-shipping-loop
description: 把一个能跑的原型迭代成"能交付、能发布"的浏览器游戏：用可判定探针守住回归、按用户原话定位手感/性能/适配问题、控制单文件体积、做中英双语、建 GitHub/Gitee 仓库并开 Pages。当用户说「继续/开工/交付/做成游戏/手机端玩不了/卡顿/体积太大/开源发布/署名」且对象是单文件 HTML 游戏时使用。
whenToUse: 用户要求把网页游戏从能跑推进到可交付发布，或报告手机端不可玩、首帧卡顿、体积过大、需要开源上架时
---

# 单文件网页游戏：交付迭代与发布

把"能玩的原型"变成"可交付的游戏"，靠的不是加功能，而是**每轮都有可判定证据**、**每类问题单独一轮**、**体积与发布路径可复现**。

## 触发条件

满足任一即执行，不要先问：

- 「继续 / 开工 / 做 / 交付 / 最后给我交付」且上下文是一个网页游戏
- 具体故障原话：「手机端完全玩不了」「横屏展示不全」「竖屏玩不了」「一进游戏就暂停」「空格失效」「E 技能有时候卡顿」「FPS 有点低」
- 内容要求：「加技能/加敌人/地图范围大/加高物/要真实物理」
- 交付要求：「压缩体积」「代码开源」「做 GitHub Gitee README」「至少一张截图」「署名用 X」「自己创建仓库」
- 用户抱怨「我要的不是玩具，是可以玩的游戏」→ 一次性补齐游戏流程（暂停/结算/最佳纪录/引导），不要反问要哪几项

## 前置闸门

进入任何改动前必须按顺序通过，不通过就先修闸门：

1. **基线可判定**：`?selftest=1` 能打印 JSON，且 `errors: 0`。跑不出来说明当前构建已坏，先修，不要叠加新功能。
2. **构建链完整**：确认打包脚本读的是**刚编译产物**（见红线第 1 条），否则改了等于没改。
3. **一轮一类**：本轮只动 手感 / 画面 / 性能 / 适配 / 内容 / 发布 中的**一类**；混着改无法归因。
4. **发布前三件套**：作者署名、LICENSE、至少一张真实截图；缺任一不许说"已发布"。
5. 用户明确要求"一次性全部做完"时，闸门 3 让步，但**验收必须逐项列出**，不能合并成一句"都好了"。

## 资源清单

| 用途 | 路径 / 命令 | 注意 |
| --- | --- | --- |
| 游戏源码（开发态） | `<项目>/_build/game.js` | 唯一真源，改这里 |
| 页面外壳（HUD/卡片/CSS） | `<项目>/_build/shell.html` | 文本节点要打 `data-i18n` |
| 引擎打包入口 | `<项目>/_build/entry-trim.js` | **按名导入**用到的 three 类，才能在 tree-shake 生效 |
| 未压缩构建 | `node build.mjs` → `index-plain.html` | 老浏览器兜底 + 静态审计对象 |
| 自解压构建 | `node build-packed.mjs` → `index.html` | gzip+base64+`DecompressionStream` |
| 源码级静态检查 | `node <skills>/single-file-3d-game/checks/check-3d-game.mjs <dir>` | 只对**未压缩**产物有效 |
| 运行时冒烟 | `powershell -File <skills>/single-file-3d-game/checks/smoke-shot.ps1 -Dir <dir> -BudgetMs 18000` | 空白图 < 15 KB 视为失败 |
| 技能校验器 | `node <skills>/dsh-skill-forge/validate-skill.mjs <SKILL.md>` | 退出码非 0 即有 ERROR |
| 建仓与 Pages | `gh repo create <name> --public --source=. --remote=origin --push`、`gh api -X POST repos/<owner>/<repo>/pages -f 'source[branch]=main' -f 'source[path]=/'` | 仓库根放一份 `index.html` 才能开 Pages |
| 托管凭据 | `python <secrets>/secrets.py --names` 查名字，`--require <KEY> --run -- <cmd>` 带值执行 | 值不落盘、不打印 |

## 标准作业流程

1. **复现并读现场**：抓一次 `?selftest=1` 的 JSON 与一张截图；用户说"卡/玩不了"时，先确认 `errors` 与探针值是否正常，避免在已坏构建上改。
2. **定位**：页面只剩标题卡、FPS 恒 0、探针不打印 → 初始化被 try/catch 吞了。临时把异常写进 `document.title`（`INIT-ERR` / `ERR:` 前缀），再 `--dump-dom` 读 `<title>`，一次拿到精确抛错点与文件行。
3. **改一类**：手感（冲量/冷却/缓冲/相机）、性能（draw call/材质/分辨率）、适配（视口/输入方案）、内容（关卡/敌人/道具）分开做。
4. **重建（必须两步）**：
   ```bash
   npx esbuild game.js --minify --target=es2020 --outfile=game.min.js
   node build.mjs && node build-packed.mjs
   ```
5. **三档验证**：静态检查（未压缩产物）→ 运行时冒烟 → `?selftest=1` 探针；三者都过才算改对。
6. **截图取证**：至少 桌面 / 手机横屏 / 手机竖屏（若声称支持）各一张；结构改动附一张"改动前 vs 改动后"的对照（FPS、draw call、体积）。
7. **记账**：把本轮改动的**根因 → 修法 → 实测数字**写进 `SPEC.md`，含踩过的坑；没有数字的"优化"不算优化。
8. **发布**：仓库根放可玩的 `index.html` + `dist/` 两份构建 + `src/` + `tools/`；`gh repo create --push`；开 Pages；回填 homepage 与 topics；最后用**线上 URL** 截一张图证明真的能玩。

## 纪律红线

- ⛔ 改了 `game.js` 却只跑 self-extract 打包（读的是旧 `game.min.js`）——先 minify 再打包，否则改动不可见。
- ⛔ 用 `stopPropagation()` 拦浏览器快捷键：它会连带吞掉游戏自己的按键（空格跳变砖）。只 `preventDefault()`。
- ⛔ 每发一颗火球就 `new PointLight() + scene.add()`：灯光数量变化会让**全场景材质重编译**，首帧必卡。用常驻灯 + 对象池 + 启动 `renderer.compile(scene, camera)`。
- ⛔ 把整座城市塞进一个 `InstancedMesh`：实例化会失去逐对象视锥剔除，帧率反降。按 300 m 分块，每块一个 InstancedMesh 并 `computeBoundingSphere()`。
- ⛔ 在真实关卡里跑探针：地形会污染判定。所有探针在 300 m 高空的无干扰测试台执行，结束后截断碰撞体、复位玩家。
- ⛔ 命名撞车：翻译函数别叫 `t`（`var t = ...` 会遮蔽它）；短名要么加前缀，要么全局唯一。
- ⛔ PowerShell 5.1 的 `.ps1` 写成非 ASCII：会被按 GBK 读成乱码导致语法错误。脚本只写 ASCII，中文交给 HTML/JS。
- ⛔ 把"看起来没问题"当验收：验收必须是命令 + 期望输出（JSON 字段、HTTP 200、字节数、PASS n/n）。
- ⛔ 声称"已发布"却没有：线上 URL 返回 200、`gh repo view` 可见、截图来自线上页面。

## 落库与验收

每次交付把下面四项落进项目内的 `SPEC.md`（或 `docs/`），缺一项视为未完成：

| 字段 | 期望 |
| --- | --- |
| 自检 JSON | 含 `errors: 0` 与关键探针（拉力米数、跳跃高度、命中/爆炸、破坏数量、输入拦截数、设备识别） |
| 静态检查 | `PASS n/n`（对未压缩产物） |
| 运行时冒烟 | 截图字节 > 15 KB（非空白），FPS 读数在屏 |
| 体积 | raw 与 brotli 两个数字，且写出本次压缩步骤 |

发布验收（真跑）：

```bash
gh repo view <owner>/<repo> --json name,url,visibility,homepageUrl,repositoryTopics
gh api repos/<owner>/<repo>/pages/builds/latest --jq '.status'      # 期望 built
curl -sI https://<owner>.github.io/<repo>/ | head -1                # 期望 200
```

## 降级与排障

| 症状 | 处理 | 必须声明 |
| --- | --- | --- |
| 页面只剩标题卡 / FPS 恒 0 | 异常被吞：写 `document.title` 后 `--dump-dom` 读回来 | 说明是初始化异常而非渲染问题 |
| 截图与改动不符、哈希完全相同 | headless 缓存或构建产物未更新：换唯一文件名、加版本 query、核对产物字节数 | 说明是哪一种，不要归因给代码 |
| 控件在 headless 里被裁、真机正常 | 布局视口 ≠ 截图尺寸（缩放因子）：把控件单侧锚定（靠左或靠右一列），并用 `visualViewport` 读数 | 声明"headless 与真机存在视口差，需真机复核" |
| 自解压版在旧浏览器白屏 | 保留 `index-plain.html`，引导页提示打开它 | 声明 `DecompressionStream` 版本门槛 |
| 用户报真机 FPS 低 | 先关反射 → 关 SSAO → 关 bloom 的分阶段降级，再谈降分辨率 | 声明降级的触发阈值与恢复方式 |
| 托管方 API 403（如 Gitee 建仓） | token 权限不足：本地配好 remote，给出一行 `git push` 与网页建仓步骤 | 明确"代码已就绪，卡在凭据权限"，不要含糊说"已发布" |
| 署名/作者变更 | 改 LICENSE + `package.json` + README 署名行，重写提交作者，再推 | 声明旧提交作者未改写（如需改写要 rebase） |

## 相关技能

- `single-file-3d-game`：单文件 three.js 场景/游戏从零搭建与资产内联，本技能的上游。
- `single-file-web-game`：Canvas 2D / 轻量 three 单文件小游戏模板。
- `dsh-skill-forge`：本技能的格式与验收来源（八段骨架 + validate-skill.mjs）。
