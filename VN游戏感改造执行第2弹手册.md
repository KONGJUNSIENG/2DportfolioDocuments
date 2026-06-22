# VN 游戏感浏览体验 — 改造执行手册 第 2 弹：Intro Scenario

> **创建日期**：2026-06-19
> **最后更新**：2026-06-22
> **状态**：✅ **Phase 1-6 完成**
> **触发**：新增 VN 情景序幕，访客必须先游玩情景再进入首页
> **品牌调性**：日系萌系 × 浅色科技感，开幕即沉浸
> **相关文档**：4.1 / 4.2 / 5.2 / [[VN游戏感改造执行手册]]

---

## 一、改造背景与目标

### 1.1 问题

当前网站访客直接进入 `/[lang]` 首页，缺少"前奏"——无角色引导、无世界观建立，首屏即展示所有内容，缺乏叙事层次的递进。

初次 VN 改造后（第 1 弹）访客进入 `/[lang]/intro` 直接播放 VN 情景，但仍缺少**标题画面**、**开场动画**和**转场动画**，体验不够"游戏化"。

### 1.2 目标

让网站访问变成"开场载入动画 → 标题画面 → 暗転转场 → VN 情景 → 加载转场 → 首页"的五段式体验：

| 游戏元素     | →   | 改造映射                                                |
| ------------ | --- | ------------------------------------------------------- |
| 开场载入     | →   | LoadingRipple (2s) → 白屏 (0.5s) → PixelsCurtain 揭露     |
| 标题画面     | →   | 着陆点先播开场动画，然后显示 TitleScreen                 |
| Start 触发   | →   | 标题文字 scramble 展开后可点击（替代 Start 按钮）        |
| Skip 按钮    | →   | 右下角，延迟 2.5s 后才出现，跳过 VN 情景直接进入首页      |
| 暗転转场     | →   | 标题→情景之间用 700ms Fade to Black 过渡                |
| 加载画面     | →   | 情景结束→首页之间用 Curtain Wipe 过渡动画               |
| 语言切换     | →   | Settings 弹窗内 radio 选择，Apply 后切换语言             |

### 1.3 设计约束

- VN 情景由 JSON 数据驱动，不硬编码场景逻辑
- VN 对话/选项使用内联渲染，不依赖 DialogBox / OptionButton 组件
- 按钮样式不使用素材图片（`option-button.png`），纯 CSS 实现游戏 UI 风格
- 全三语支持（zh / ja / en）
- 所有动画仅依赖 Framer Motion，不引入新库
- 角色立绘通过场景 JSON 的 `characterImage` 字段控制，支持按场景切换

---

## 二、改动全景图

### 2.1 新增文件（17 个）

| #   | 文件                              | 用途                                                                 |
| --- | --------------------------------- | -------------------------------------------------------------------- |
| 1   | `src/data/vn-scenario.zh.json`      | ✅ VN 中文剧本（43 场景，多分支 + 终场 + repel 排斥场景）               |
| 2   | `src/data/vn-scenario.ja.json`      | ✅ VN 日文剧本                                                         |
| 3   | `src/data/vn-scenario.en.json`      | ✅ VN 英文剧本                                                         |
| 4   | `src/components/VNScenario.tsx`     | ✅ VN 情景播放器核心组件（状态机驱动，逐帧播放场景）                    |
| 5   | `src/components/IntroShell.tsx`      | ✅ 客户端状态机，管理 5 阶段流转（intro → transition → scenario → loading） |
| 6   | `src/components/IntroAnimation.tsx`  | ✅ 开场动画编排组件：ripple → white → curtains → title 序列            |
| 7   | `src/components/LoadingRipple.tsx`   | ✅ 三环涟漪脉冲载入动画（~2.0s 自动完成）                               |
| 8   | `src/components/PixelsCurtain.tsx`   | ✅ 8×8 像素块随机缩放揭露动画                                          |
| 9   | `src/components/ScrambleText.tsx`    | ✅ 文字 scramble-on-hover 组件：idle → scrambling → revealed → reverting |
| 10  | `public/images/bigsmile.png`         | ✅ 角色表情立绘：大笑                                                    |
| 11  | `public/images/emm.png`              | ✅ 角色表情立绘：无语                                                    |
| 12  | `public/images/mad.png`              | ✅ 角色表情立绘：生气                                                    |
| 13  | `public/images/normal.png`           | ✅ 角色表情立绘：普通                                                    |
| 14  | `public/images/sad.png`              | ✅ 角色表情立绘：伤心                                                    |
| 15  | `public/images/shock.png`            | ✅ 角色表情立绘：震惊                                                    |
| 16  | `public/images/shy.png`              | ✅ 角色表情立绘：害羞                                                    |
| 17  | `public/images/smile.png`            | ✅ 角色表情立绘：微笑（5 场景使用）                                       |

### 2.2 修改文件（8 个）

| #   | 文件                              | 改动                                                                   |
| --- | --------------------------------- | ---------------------------------------------------------------------- |
| 1   | `src/proxy.ts`                      | ✅ 默认重定向 `/` → `/zh/intro`                                         |
| 2   | `src/components/LayoutShell.tsx`     | ✅ intro 路由下隐藏 Navbar/Footer                                       |
| 3   | `src/lib/read-data.ts`              | ✅ 追加 `getVNScenario()` + Zod schema（含 `nextScene`/`repel`/`autoAdvanceMs` 字段） |
| 4   | `src/app/[lang]/intro/page.tsx`      | ✅ 改为只读数据 → 渲染 `<IntroShell>`（Server → Client bridge）         |
| 5   | `src/components/TitleScreen.tsx`     | ✅ **重写** 改为 centercard 布局（✦装饰线+粉辉光），移除角色立绘和 Start 按钮，标题改用 ScrambleText，新增 Settings 弹窗代替语言 pill 按钮，Skip 右下角延迟显示 |
| 6   | `src/components/LoadingOverlay.tsx`  | ✅ **重写** curtain wipe 动画（左右面板滑动展开），替代 loading 文字     |
| 7   | `src/components/VNScenario.tsx`      | ✅ 选项居中 overlay + bg-black/50 遮罩 + stagger 入场；人物尺寸 108vh/136vh；对话内联渲染底部；新增 `nextScene` 跳转、`isEnding` 检测、**鼠标排斥交互**、**自动推进倒计时**、**点击锁**、字体加大 |
| 8   | `public/images/hero-character.png`   | ✅ 立绘素材更新（替换原始 C1-1.png）                                      |

### 2.3 删除文件

| 文件                             | 原因                                               |
| -------------------------------- | -------------------------------------------------- |
| `public/images/dialog-box.png`   | 不再使用（VNScenario 改用内联渲染，顶替 emotion 立绘） |

### 2.4 无需改动

| 文件                                 | 原因                                       |
| ------------------------------------ | ------------------------------------------ |
| `src/components/DialogBox.tsx`         | VNScenario 不使用，对话内联渲染             |
| `src/components/OptionButton.tsx`      | VNScenario 不使用，选项内联渲染             |
| `src/components/ChapterHeader.tsx`     | VN 情景内不使用章节标题                    |
| `src/components/SectionTransition.tsx` | 不涉及章节过渡                             |
| `src/app/[lang]/page.tsx`              | 首页结构完全不变                           |
| `src/styles/globals.css`               | 无需新增 CSS 变量                          |
| `src/lib/animations.ts`                | 追加 VN variants（chapterReveal/dialogSlideUp/optionStagger/optionItem/transitionFade） |
| `src/app/[lang]/layout.tsx`            | 不修改（判断逻辑下沉到 LayoutShell）         |

---

## 三、流程架构

### 3.1 状态机流程

```
/[lang]/intro 着陆
       │
       ▼
┌────────────────┐
│  IntroAnimation │  ← 多阶段开场动画序列
│                 │
│  ① LoadingRipple│── 三环涟漪脉冲 (2.0s)
│  ② 白屏闪烁    │── 纯白画面 (0.5s)
│  ③ PixelsCurtain│── 8×8 方块随机揭露
│  ④ TitleScreen  │── 动画完成后显示标题画面
└────────────────┘
       │
       ▼
┌────────────────────────┐
│  TitleScreen            │  ← 场景标题画面（centered card 布局）
│  [▲ 文字可点击触发]      │──→ 暗転转场 (700ms Fade to Black) → VNScenario
│  [Skip (右下角, 2.5s后)]│──→ IntroShell "loading" → LoadingOverlay → router.push
│  [▶ Settings]           │──→ 语言切换弹窗 (radio + Apply)
└────────────────────────┘
       │
       ▼ (click title)
 暗転过渡阶段 (700ms)
  → Title exit fade (300ms)
  → Black fade in (300ms)
  → setTimeout(700ms) → phase = "scenario"
  → Black fade out (300ms)
  → Scenario fade in (300ms)
       │
       ▼
┌─────────────────┐
│  VNScenario     │  ← 播放剧本剧情 + 内联对话 + 选项
│  [终场]         │──→ onFinish → IntroShell "loading"
└─────────────────┘
       │
       ▼
┌─────────────────┐
│  LoadingOverlay  │  ← Curtain Wipe 动画 (左右面板展开, 1.4s)
│  → router.push  │
└─────────────────┘
       │
       ▼
 ┌───────────┐
 │  /[lang]   │  ← 首页（Navbar/Footer 恢复正常）
 └───────────┘
```

### 3.2 组件树

```
intro/page.tsx (Server Component, 读取 vn-scenario JSON)
  └── IntroShell (Client Component, 状态机)
      ├── IntroAnimation (phase === "intro")
      │   ├── LoadingRipple (三环涟漪, ~2s)
      │   ├── 白屏 (0.5s)
      │   ├── PixelsCurtain (8×8 方块揭露)
      │   └── TitleScreen (after 动画完成)
      │       ├── ScrambleText (hover 乱码 → revealed 可点击 → 触发 onStart)
      │       ├── 装饰：✦ 线条 + 粉辉光 + 圆角边框
      │       ├── 副标题 "Software Developer Portfolio"
      │       ├── [▶ Settings] 按钮 (居中, ghost style)
      │       ├── [Skip >>] (右下角, 2.5s 后 animate 淡入, ghost style)
      │       └── Settings 弹窗 (language radio + Apply/Close, ESC to close)
      │
      ├── 暗転遮罩 (phase === "transition", 700ms, 纯黑 bg)
      │
      ├── VNScenario (phase === "scenario")
      │   ├── 角色立绘 (108vh/136vh, 通过 characterImage 字段切换)
      │   ├── 底部对话区域 (内联渲染, no DialogBox, h-[30vh])
      │   ├── 选项 overlay (居中, bg-black/50 遮罩, stagger 入场, text-xl md:text-2xl)
      │   ├── 鼠标排斥交互 (repel, 选项组整体逃离光标, CSS 0.15s ease-out)
      │   ├── 自动推进 (autoAdvanceMs 倒计时 → 自动选首个选项)
      │   ├── 点击锁 (auto-advance 后 500ms 阻挡 advance, 防连点跳过)
      │   └── 终场 → onFinish
      │
      └── LoadingOverlay (phase === "loading", curtain wipe 动画)
          ├── 左面板 (from 50% → 0% width, bg-[#1A1A2E])
          ├── 右面板 (from 50% → 0% width, bg-[#1A1A2E])
          └── → onAnimationComplete → router.push(/${lang})
```

---

## 四、TitleScreen 设计

### 4.1 布局

```
┌─────────────────────────────────────┐
│      背景: 浅色渐变 + vignette 遮罩    │
│              + 粉辉 glow             │
│                                     │
│   ✦ ───── ✦ ───── ✦                 │  ← 顶部装饰线
│                                     │
│         KONG JUN SIENG              │  ← 标题 (ScrambleText, hover 乱码)
│         (hover → 开始游戏 /          │
│           スタート / START)          │
│                                     │
│   ✦ ───── ✦ ───── ✦                 │  ← 底部装饰线
│       ───── (蓝线 divider) ─────      │
│       Software Developer Portfolio  │  ← 副标题
│                                     │
│           [▶ 设定]                 │  ← Settings 按钮 (▶ 符号, 居中, ghost)
│                                     │
│                    [▶ Skip] (右下角)  │  ← 2.5s 后淡入
│                                     │
│                    v1.0              │  ← 版本号 (底部居中)
└─────────────────────────────────────┘
```

### 4.2 按钮样式（纯 CSS，无素材）

| 按钮       | 样式                                                                                   |
| ---------- | -------------------------------------------------------------------------------------- |
| **Settings** | ghost style: 无背景, `text-text-secondary`, hover 变色 + x 偏移动效                     |
| **Skip**     | ghost style, 右下角 absolute 定位, animate 淡入, `text-sm`, hover 变色                  |
| **Settings 弹窗** | `bg-surface/95` + `border-dialog-blue/30` rounded-xl, blur backdrop, 居中 overlay      |

### 4.3 ScrambleText 交互状态机

标题文字 "KONG JUN SIENG" 使用 ScrambleText 组件：

| 阶段           | 触发条件         | 行为                                                                 |
| -------------- | ---------------- | -------------------------------------------------------------------- |
| idle           | 初始状态         | 显示 "KONG JUN SIENG"                                                |
| scrambling     | mouse enter      | 40ms 间隔随机字符替换，~800ms 内从左到右稳定为目标文字                 |
| revealed       | scramble 完成    | 显示 locale 目标文字，"KONG JUN SIENG" → "开始游戏"/"スタート"/"START" |
| click (revealed)| 用户点击        | 触发 `onStart` → 暗転转场                                             |
| leaving (revealed)| mouse leave   | 100ms 后自动 revert                                                   |
| reverting      | leave 超时 / 打断| 字符乱码→还原到 "KONG JUN SIENG"                                     |
| attract        | idle 时 2.5s 周期 | 自动 1.5s scramble→0.5s hold→1s revert，提示用户可交互（期间 click 被阻止）|

### 4.4 交互行为

| 操作                    | 行为                                                 |
| ----------------------- | ---------------------------------------------------- |
| hover 标题文字          | scramble 到 locale 文本（开始游戏/スタート/START）   |
| hover 标题文字(已revealed)| 保持 revealed，不启动倒计时                          |
| 点击标题文字 (revealed) | 触发暗転转场 → 进入 VNScenario                        |
| 鼠标从标题离开 (revealed)| 100ms 后自动 revert 到 "KONG JUN SIENG"              |
| 鼠标中途 re-enter       | 取消倒计时，保持 revealed                             |
| 点击 ▶ Settings         | 打开 Settings 弹窗（language radio + Apply/Close）    |
| Settings Apply          | `router.replace(/${selectedLang}/intro)` 切换语言     |
| ESC / 点击 overlay      | 关闭 Settings 弹窗                                    |
| 点击 Skip (右下角)      | 直接跳转到 LoadingOverlay → 首页                       |

---

## 五、暗転转场设计

使用 Framer Motion `AnimatePresence mode="wait"` + `setTimeout` 控制时序：

```
t=0ms     点击 title (revealed) → phase 切换为 "transition"
           → IntroAnimation/TitleScreen exit (opacity 1→0, 300ms)
t=300ms   Title 完全退出 → 黑屏遮罩 enter (opacity 0→1, 300ms)
t=600ms   黑屏完全显示
t=700ms   setTimeout 触发 → phase 切换为 "scenario"
           → 黑屏遮罩 exit (opacity 1→0, 300ms)
           → VNScenario enter (opacity 0→1, 300ms)
t=1000ms  VNScenario 完全显示
```

总时长 ≈ 1.0s（黑屏 phase 持续 700ms）。纯 Framer Motion 实现，不引入额外库。

---

## 六、VNScenario 增强设计

### 6.1 选项居中

当 `currentScene.choices` 存在时，选项渲染在画面**正中央**：

```
┌────────────────────────────┐
│                             │
│     [角色立绘 (背景暗化)]   │
│      (bg-black/50 遮罩)     │
│                             │
│   ┌─────────────────────┐   │  ← 居中 overlay, stagger 入场
│   │  ▶ 选项A (text-xl)   │   │
│   │  ▶ 选项B (md:text-2xl)│   │
│   └─────────────────────┘   │
│                             │
│  ┌───────────────────────┐  │
│  │ 对话 (内联渲染, no     │  │  ← 底部，保持可见
│  │ DialogBox 组件)        │  │
│  └───────────────────────┘  │
└────────────────────────────┘
```

### 6.2 人物尺寸

| 断点     | 实际值      | 说明                 |
| -------- | ----------- | -------------------- |
| Mobile   | **108vh**   | 超出屏幕，营造贴近感  |
| Desktop  | **136vh**   | 大幅放大角色存在感    |

人物尺寸远超原文档的 +25% 方案，通过 `top-[-50]` 偏移使角色腰部以下被裁切，产生 VN 式"大立绘贴近"效果。

### 6.3 对话渲染

不使用 DialogBox 组件。对话和角色名直接内联渲染在底部 overlay（`h-[30vh]` 渐变遮罩区域），Framer Motion `AnimatePresence mode="wait"` 控制场景切换过渡。

### 6.4 选项样式

| 项目       | 规格                           |
| ---------- | ------------------------------ |
| 字体大小   | `text-xl md:text-2xl`           |
| 文字颜色   | `text-white/80` → hover `text-white` |
| 入场动画   | stagger 0.1s, 左移 16px → 原位     |
| 指示器     | ▶ 图标，hover 时 opacity 0→1, 右移  |

### 6.5 场景数据驱动

VN 场景由 JSON 数据定义，支持字段：

| 字段             | 类型       | 用途                                   |
| ---------------- | ---------- | -------------------------------------- |
| `id`             | string     | 场景唯一标识，用于分支跳转             |
| `characterName`  | string|null| 角色名（null 时不显示）                |
| `dialogue`       | string     | 台词文本                               |
| `choices`        | array|null | 选项列表（null/空时点击推进）          |
| `nextScene`      | string?    | 非选项场景的跳转目标，跳过中间场景（如场景5→8） |
| `repel`          | boolean?   | 启用鼠标排斥交互（选项整体逃离光标）    |
| `autoAdvanceMs`  | number?    | 自动推进毫秒数，倒计时结束后自动选择首个选项（同时启用点击锁 500ms） |
| `isEnding`       | boolean    | 终场标记，结束后调 `onFinish`          |
| `characterImage` | string?    | 角色立绘路径（可选，缺省时显示默认立绘），支持按场景切换 |

### 6.6 终场路由控制

VNScenario 不再直接调用 `router.push`，改为调用 `onFinish` 回调。由父组件 IntroShell 决定后续处理（进入 LoadingOverlay）。

### 6.7 鼠标排斥交互（Repel）

当 `repel: true` 时，选项 overlay 上的鼠标移动会触发排斥效果：

| 项目         | 规格                                                              |
| ------------ | ----------------------------------------------------------------- |
| 触发         | `onPointerMove` 在 overlay `<div>` 上                              |
| 算法         | RAF 内计算鼠标到 overlay 中心的距离与方向，反转并缩放为偏移量     |
| 阈值         | 200px（距离 > 200px 时不排斥）                                    |
| 偏移公式     | `strength = ((threshold - dist) / threshold) * 1000`；`offset = -(dx / dist) * strength` |
| 过渡         | CSS `transform 0.15s ease-out`（wrapper div）                     |
| 离开复位     | `onPointerLeave` 时 `repelOffset` 归零                            |
| 场景切换     | `advance`/`choose` 中 `setRepelOffset({x:0,y:0})` 复位            |

```
鼠标靠近选项区域 → 整个选项组（wrapper div）以 transform:translate 逃离
               → dist 趋近 5px 时偏移可达~1000px，确保用户无法点中选项
               → 鼠标距离超过 200px 时偏移归零
```

### 6.8 自动推进 + 点击锁（Auto-Advance & Click Lock）

当 `autoAdvanceMs` 存在时：

| 项目         | 规格                                                              |
| ------------ | ----------------------------------------------------------------- |
| 倒计时       | `setTimeout` 在场景挂载时启动，时长 `autoAdvanceMs`                |
| 触发         | 到时自动调用 `choose(choices[0].nextScene, ...)`                  |
| 点击锁       | 自动推进后设置 `advanceLockedRef = true`，2000ms 后解锁            |
| 锁的作用     | 阻挡 `advance()` 执行，防止用户连点导致跳过后续剧情               |
| 锁清理       | 场景卸载时 `advanceLockedRef = false` + `clearTimeout`             |

```
t=0ms     场景 37/39 挂载 → setTimeout(5000)
t=5000ms  choose(首个选项) → advanceLocked = true
t=7000ms  advanceLocked = false → advance() 恢复正常
```

---

## 七、LoadingOverlay 设计

### 7.1 Curtain Wipe 动画

替代原"Now Loading 文字 + 三点脉冲"方案，使用双面板 curtain wipe 效果：

```
t=0ms       LoadingOverlay mount
           → opacity=1 (瞬时)
t=600ms     delay 结束, 动画开始
           → 左面板 width: 50% → 0%
           → 右面板 width: 50% → 0%
t=1400ms    面板完全缩回
           → onAnimationComplete 触发 router.push(/${lang})
```

| 项目         | 规格                                                     |
| ------------ | -------------------------------------------------------- |
| 背景色       | `bg-[#1A1A2E]` 深海军蓝                                  |
| 动画         | Framer Motion `animate={{ width: "0%" }}`                |
| 延迟         | 0.6s delay                                                |
| 时长         | 0.8s cubic-bezier 缓出                                    |
| 跳转触发     | `onAnimationComplete` 回调，防重复通过 `navigated ref`     |

无需额外 JS 或 CSS，纯 Framer Motion 实现。

---

## 八、执行任务表

### ✅ 已完成（VN2 Phase 1：初版实现）

| ID      | 任务                             | 状态 |
| ------- | -------------------------------- | :--: |
| VN2-1.1 | 创建 vn-scenario.zh.json         |  ✅  |
| VN2-1.2 | 创建 vn-scenario.ja.json         |  ✅  |
| VN2-1.3 | 创建 vn-scenario.en.json         |  ✅  |
| VN2-1.4 | read-data.ts 追加 Schema + 读取函数 |  ✅  |
| VN2-3.1 | 创建 intro/page.tsx（初版）    |  ✅  |
| VN2-3.2 | 修改 proxy.ts 重定向             |  ✅  |
| VN2-3.3 | 修改 LayoutShell.tsx 隐藏导航    |  ✅  |
| VN2-4   | lint + typecheck + build 验证    |  ✅  |

### ✅ 已完成（VN2 Phase 2：TitleScreen + 开场动画 + 转场系统）

| ID       | 任务                                                        | 状态 |
| -------- | ----------------------------------------------------------- | :--: |
| VN2-5.1  | 创建 ScrambleText.tsx（scramble-on-hover 文字组件）          |  ✅  |
| VN2-5.2  | 创建 LoadingRipple.tsx（三环涟漪载入动画）                    |  ✅  |
| VN2-5.3  | 创建 PixelsCurtain.tsx（8×8 像素块揭露动画）                 |  ✅  |
| VN2-5.4  | 创建 IntroAnimation.tsx（ripple→white→curtains→title 编排）   |  ✅  |
| VN2-5.5  | 重写 TitleScreen.tsx（centered card、ScrambleText、Settings 弹窗、Skip 延迟） |  ✅  |
| VN2-5.6  | 重写 LoadingOverlay.tsx（curtain wipe 替代 loading 文字）     |  ✅  |
| VN2-5.7  | 修改 IntroShell.tsx（phase "title" → "intro"，适配新状态机） |  ✅  |
| VN2-5.8  | 修改 intro/page.tsx → 渲染 IntroShell                       |  ✅  |

### ✅ 已完成（VN2 Phase 3：VNScenario 增强）

| ID       | 任务                                                        | 状态 |
| -------- | ----------------------------------------------------------- | :--: |
| VN2-6.1  | 选项移至画面正中央（overlay + stagger 入场）                  |  ✅  |
| VN2-6.2  | 人物尺寸放大至 108vh/136vh（超大立绘贴近效果）                 |  ✅  |
| VN2-6.3  | 对话内联渲染（不依赖 DialogBox 组件）                         |  ✅  |
| VN2-6.4  | 选项文字大小调整（text-lg md:text-xl）                        |  ✅  |
| VN2-6.5  | 终场调用 onFinish 回调替代 router.push                       |  ✅  |

### ✅ 已完成（VN2 Phase 4：验证）

| ID       | 任务                                     | 状态 |
| -------- | ---------------------------------------- | :--: |
| VN2-7.1  | `pnpm lint`                              |  ✅  |
| VN2-7.2  | `pnpm typecheck`                         |  ✅  |
| VN2-7.3  | `pnpm build`                             |  ✅  |
| VN2-7.4  | 全流程测试：标题→暗転→情景→选择→加载→首页 |  ✅  |

### ✅ VN2 Phase 5：剧情改写 + 根据场景切换立绘

| ID       | 任务                                                    | 状态 |
| -------- | ------------------------------------------------------- | :--: |
| VN2-8.1  | 重写 vn-scenario.*.json 三语剧本内容（43 场景新剧情）      |  ✅  |
| VN2-8.2  | 将立绘素材放入 `public/images/` （bigsmile/emm/mad/normal/sad/shock/shy/smile） |  ✅  |
| VN2-8.3  | Zod `VNSceneSchema` 追加 `characterImage` 字段            |  ✅  |
| VN2-8.4  | 更新 JSON 数据 → 使用 `characterImage` 指定各场景立绘     |  ✅  |
| VN2-8.5  | 验证立绘切换：对话推进/选项选择后立绘正确更新             |  ✅  |
| VN2-8.6  | 全流程 lint + typecheck + build                           |  ✅  |

### ✅ VN2 Phase 6：repel 交互 + 自动推进 + 点击锁 + 剧情字号调整

| ID       | 任务                                                    | 状态 |
| -------- | ------------------------------------------------------- | :--: |
| VN2-9.1  | Zod `VNSceneSchema` 追加 `nextScene`/`repel`/`autoAdvanceMs` 字段 |  ✅  |
| VN2-9.2  | VNScenario 实现 `nextScene` 跳转（非选项场景跳过中间序列）  |  ✅  |
| VN2-9.3  | VNScenario 实现 `isEnding` 主动检测（支持多终场）          |  ✅  |
| VN2-9.4  | VNScenario 实现鼠标排斥交互（repel, RAF + transform 逃离） |  ✅  |
| VN2-9.5  | VNScenario 实现自动推进倒计时 + 点击锁（防连点跳过）       |  ✅  |
| VN2-9.6  | 整站字号加大：对话 text-xl→2xl, 角色名 text-base→lg       |  ✅  |
| VN2-9.7  | dialog 区域扩高 h-[25vh]→h-[30vh]                         |  ✅  |
| VN2-9.8  | 修复选项按钮 key 重复（choice.nextScene → choice.text）    |  ✅  |
| VN2-9.9  | 同步 vn-scenario.en.json / vn-scenario.ja.json 为新结构   |  ✅  |
| VN2-9.10 | 全流程 lint + typecheck + build                           |  ✅  |

---

## 九、验收清单

### 开场动画
- [x] LoadingRipple 三环涟漪 2s 后自动完成
- [x] 白屏闪烁 0.5s
- [x] PixelsCurtain 8×8 方块随机缩放揭露
- [x] 动画完成后正常显示 TitleScreen

### 标题画面
- [x] 访问 `/` 自动重定向到 `/zh/intro` → 开场动画 → TitleScreen
- [x] 标题「KONG JUN SIENG」+ 副标题显示正常
- [x] hover 标题时 scramble 到 locale 目标文字
- [x] 点击 revealed 标题 → 暗転转场开始
- [x] 鼠标从标题离开 → 100ms 后自动 revert
- [x] 每 2.5s 自动 scramble flash 提示（attract 模式，期间 click 被阻止）
- [x] Skip 按钮：右下角，2.5s 后淡入
- [x] 点击 Skip → 跳转到 LoadingOverlay → 首页
- [x] ▶ Settings 按钮：打开弹窗，language radio 选择，Apply 切换语言
- [x] Settings ESC / overlay 点击可关闭

### 暗転转场
- [x] Title exit fade out (300ms)
- [x] 黑屏 fade in (300ms)
- [x] 700ms setTimeout → phase 切换
- [x] 黑屏 fade out + VNScenario fade in

### VN 情景
- [x] 角色立绘 108vh/136vh 超大尺寸
- [x] 无选项时点击推进场景（支持 nextScene 跳过中间序列）
- [x] 有选项时：选项居中 + 角色区域 bg-black/50 overlay
- [x] 选项文字 text-xl md:text-2xl
- [x] 选项 stagger 入场（0.1s 间隔）
- [x] 选项选择后进入对应分支
- [x] 终场 → onFinish → LoadingOverlay
- [x] 场景切换时 characterImage 变换 → 立绘 Fade Motion 过渡
- [x] 多终场支持（场景 25/29/32/35/40/43）
- [x] repel 场景：鼠标靠近时选项逃离，无法点击
- [x] autoAdvance 场景：5s 后自动推进到下一场景
- [x] 点击锁：auto-advance 后 500ms 内阻挡 advance，防连点跳过

### 加载转场
- [x] Curtain Wipe：左右面板 bg-[#1A1A2E] 滑动展开
- [x] 0.6s delay + 0.8s 动画
- [x] onAnimationComplete → 自动跳转到首页

### 布局
- [x] TitleScreen / VNScenario / LoadingOverlay 均无 Navbar
- [x] 跳转到首页后 Navbar/Footer 正常显示
- [x] 直接访问 `/[lang]` 可绕过 intro

---

## 十、回滚方案

| 层级       | 操作                                                                 |
| ---------- | -------------------------------------------------------------------- |
| **完整回滚** | 删除 `app/[lang]/intro/` + 所有新增组件（含 emotion PNG），恢复 `proxy.ts` 原始内容 |
| **仅新功能回滚** | 删除 `IntroAnimation.tsx`/`ScrambleText.tsx`/`LoadingRipple.tsx`/`PixelsCurtain.tsx`，还原旧版 TitleScreen |
| **数据级** | 删除 `src/data/vn-scenario.*.json`                                   |
| **布局级** | 恢复 `LayoutShell.tsx` 移除 intro 路由判断                            |
