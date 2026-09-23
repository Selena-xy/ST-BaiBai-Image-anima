# 柏宝绘 · SillyTavern 剧情配图助手（anima 魔改版）

基于 [baibai-git/ST-BaiBai-Image](https://github.com/baibai-git/ST-BaiBai-Image) 的 fork。原插件功能全部保留，额外为「NAI 形状的 ComfyUI 转发站」（如 latent.moe：接口是 NAI 协议、后端实际跑 anima / ComfyUI 工作流）做了一个提示词规范开关，并按实卡实测持续加固提示词规范。

## 这个 fork 改了什么

### 1. NAI 面板「提示词规范」开关

| 选项 | 行为 | 适用 |
|---|---|---|
| **NAI 规范（官方/兼容站）** | 与上游一致：Base Prompt + 原生 Character Prompts（`characters[]`） | NovelAI 官方、真 NAI 协议兼容站 |
| **ComfyUI 规范（转发站/anima）** | 产出单串 tag + 一段连贯英文自然语言（nl），**不产 `characters[]`** | 后端实际是 ComfyUI 工作流的转发站 |

**为什么需要**：上游把角色提示放进 `v4_prompt.caption.char_captions`，转发站不解析这个结构，而是把 `input` 整串塞给固定工作流——Base tag、画师串、各角色 tag/nl 全被拍平成一条混杂长串。切到 ComfyUI 规范后，实际发出的 `input` 是干净三段：`[画师串], [画面 tag], [质量词]. [自然语言]`。

**配套一致性**：切规范时思维链同步换成 ComfyUI 版；建档 nl 校验豁免；`supportsCharacters` 报 false；强制开启 nl。

### 2. 实卡实测加固（规范 / 思维链默认值）

用真实卡在酒馆连跑多楼、逐楼用视觉模型（GLM-5.3）按提示词逐项验收，按「同类问题重复 ≥3 次即修」的规则加固：

- **视线**：单人画面禁止 `looking at another`；单人看物件改用动作词 + `looking down`
- **景别**：同任务多张图必须至少两种景别；思维链 E 段 P 列表写成 `P2=核心动作@景别`
- **槽间硬差异**：各图必须能指出不同时间点 / 地点 / 人物组合 / 核心动作
- **瞳色硬事实**：tag 与档案一字不差，nl 用 `black-eyed` 式复合词复述
- **单人防路人**：公共场所近景加 `depth of field` / `blurred background` 压掉背景人物
- **接触类动作**：写明「哪只手 + 接触什么 + 完成态」（`a fork speared through the toast`、`already in her open hands`）
- **左右手**：只在剧情必需时指定（渲染模型左右翻车率高，非必需写 `his hand`）
- **悬浮物**：写明与手和地面的分离（`floating in mid-air, away from his hand`）
- **配件与版型**：发长 / 袖型 / 发夹 / 眼镜 / 道具佩戴方式必须写具体词并在 nl 复述
- **素色服装**：`plain` + `solid color`（+ 材质词、blank front），禁印花图案文字
- **文字载体**：屏幕 / 海报 / 纸盒 / 书页 / 围裙一律 `blank` / `textless` / `plain`，避免近景特写
- **面部细节**：黑眼圈、疲惫感等档案细节必须进 tag 并在 nl 复述
- **手部**：涉及手部的核心动作，整只手必须完整入画

实测（改前 → 改后）：单人误用 `looking at another` 33% → 6%；每楼景别唯一数 1.57 → 2.09；`medium shot` 占比 76% → 59%。严格验收均分：美学 4.4 / 忠实 3.2~3.7（剩余扣分集中在渲染模型能力边界：文字乱码、精细姿态、道具形态）。

### 3. 负面词层修复

负面词解析链是 `配方（画师串）绑定值 → 渠道覆盖值 → 内置默认`，**前一级非空则后面整层失效**。画师串一旦绑定自己的负面词，内置通用层（`logo, watermark, signature, text, bad hands, extra fingers ...`）会被顶掉——实测中签名水印与乱码反复出现正是这个原因。本 fork 的 5-full 通用层额外补了 `graphic print, print on clothes, text on clothes, book text`。

> 补充结论（实测）：签名水印来自画师串风格自带，停用画师串即消失；而衣物印花/伪文字属渲染模型自身先验，提示词与负面词只能压制、无法根除。

## 安装

酒馆「扩展 → 安装扩展」填入本仓库地址：

```
https://github.com/Selena-xy/ST-BaiBai-Image-anima
```

装好后进「设置 → 渠道」配置出图渠道（本地 ComfyUI 或 NovelAI 账号），确认「自动生成 tag」开关已开即可。若目标是 ComfyUI 转发站，把「提示词规范」切到 **ComfyUI 规范**。

## 与上游同步

```bash
git remote add upstream https://github.com/baibai-git/ST-BaiBai-Image.git
git fetch upstream
git merge upstream/main   # 冲突大概率在 prompt.ts / settings.ts / NaiPanel.vue / runner.ts / generate.ts
pnpm install && pnpm build
```

## 构建

```bash
pnpm install
pnpm build      # vite build + 自动同步 manifest 版本号
```

## License

与上游一致（见 LICENSE）。
