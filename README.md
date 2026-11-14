# 🚢 web-game-shipping-loop

> **把"能跑的原型"推成"能交付发布"的浏览器游戏。**
> 一个 agent skill：可判定探针守回归、按原话定位手感/性能/适配、单文件体积预算、中英双语、GitHub/Gitee 建仓开 Pages。

<p align="center">
  <img src="https://img.shields.io/badge/agent--skill-DSH%20%2F%20Claude%20Code%20%2F%20Codex-3B82F6" alt="agent skill" />
  <img src="https://img.shields.io/badge/topic-dsh--skill-8957e5" alt="topic" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-green" alt="license" /></a>
</p>

## 这是什么

一份给 agent 的**操作规程**，不是一个知识摘要。判据只有一条：把它单独丢给一个没有任何上下文的 agent，它能否不问一句就动手，并在失败时自己判定失败。

它来自一次真实的交付：从"单文件 3D 跑酷原型"一路迭代到 244 KB 可发布版本、双语 UI、GitHub 仓库 + Pages 在线可玩。

## 装

```bash
# ① git 方式（推荐，DSH 用户级、跨项目可用）
git clone https://github.com/EternalNight996/web-game-shipping-loop.git \
  ~/.dsh/skills/web-game-shipping-loop

# ② 任何读 skills 目录的 agent 宿主
npx skills add https://github.com/EternalNight996/web-game-shipping-loop

# ③ npm（包已发布，见下方状态）
npm i -D dsh-skill-web-game-shipping-loop
```

> npm 包 `dsh-skill-web-game-shipping-loop`：0.1.0 / 0.1.1 已提交，处于 npm 的 staged 审核态（新包首次发布需要账号侧确认），registry 的 `latest` 尚未指向它。**不影响 git / `npx skills add` 两种安装方式。**

## 它管什么

| 段 | 内容 |
|---|---|
| 触发条件 | 「继续 / 开工 / 交付 / 手机端玩不了 / 空格失效 / E 技能卡顿 / 压缩体积 / 开源发布 / 署名用 X」 |
| 前置闸门 | 基线探针绿 → 构建链读到新产物 → 一轮只改一类 → 发布前三件套 |
| 资源清单 | 源码 / 外壳 / tree-shake 入口 / 两种构建 / 静态检查 / 冒烟 / 校验器 / 建仓命令（绝对路径表） |
| 标准流程 | 复现 → 定位（把异常写进 document.title 再 dump-dom）→ 改一类 → minify+打包 → 三档验证 → 截图 → 记账 → 发布 |
| 纪律红线 | 9 条，每条对应一次真实事故（见下） |
| 落库验收 | 自检 JSON / PASS n/n / 截图字节 / raw+brotli 体积 / Pages 状态码 |
| 降级排障 | 标题卡空转、截图与改动不符、控件被裁、旧浏览器白屏、真机掉帧、托管 403 |

## 9 条红线（都是踩出来的）

1. 改了源码却只跑打包（读的是旧 `game.min.js`）→ 白改
2. `stopPropagation()` 会吞掉游戏自己的按键（Space 跳变砖）
3. 每发子弹 `new PointLight + scene.add` → 灯光数量变化触发**全场景材质重编译**，首帧必卡
4. 整城塞进一个 `InstancedMesh` → 丢视锥剔除，FPS 反降；按 300 m 分块
5. 探针跑在真实关卡里 → 地形污染判定；一律 300 m 高空测试台
6. 翻译函数叫 `t` → 被 `var t = ...` 遮蔽
7. PowerShell 5.1 的 `.ps1` 写非 ASCII → 按 GBK 读成乱码
8. 「看起来没问题」当验收 → 截图哈希相同却没发现构建没更新
9. 没验线上就说已发布 → 托管 403 差点被含糊过去

## 验收（可复制）

```bash
node <skills>/dsh-skill-forge/validate-skill.mjs SKILL.md   # 期望 PASS (errors=0, warns=0)
gh repo view <owner>/<repo> --json name,url,visibility,homepageUrl,repositoryTopics
gh api repos/<owner>/<repo>/pages/builds/latest --jq '.status'
```

## 相关技能

- `single-file-3d-game`：从零搭单文件 three.js 场景（上游）
- `single-file-web-game`：Canvas 2D / 轻量单文件模板
- `dsh-skill-forge`：本技能的八段骨架与校验器来源

## License

MIT © mengbot
