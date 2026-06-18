# VN 游戏感浏览体验 — 改造执行手册

> **创建日期**：2026-06-18  
> **触发**：网站浏览体验从"干净网页"升级为"视觉小说式浏览"  
> **品牌调性**：日系萌系 × 浅色科技感，Gal 社风格 + VN 交互模式  
> **相关文档**：3.1 / 3.2 / 3.3 / 4.2 / 4.3 / 5.2（均同步更新）

---

## 一、改造背景与目标

### 1.1 问题

现有页面结构是"带立绘的标准网页"——Hero 不错，但 Hero 以下的部分缺少沉浸感，浏览体验与普通 portfolio 无异。

### 1.2 目标

让浏览过程像推进视觉小说一样：

| VN 元素 | → | 网站映射 |
|---------|---|---------|
| 章节切换 | → | 每个 Section 顶部显示 CHAPTER N + 装饰分割线 |
| 对话框 | → | About 自我介绍、Case Study 描述用蓝色双层对话框承载 |
| 选择肢 | → | CTA / 查看详情 / 简历下载用选项按钮素材风格 |
| 过渡台词 | → | Section 之间出现短暂台词对话框，淡入→停留→淡出 |
| 系统菜单 | → | Contact 区块改为菜单列表样式；Navbar 增加菜单感装饰 |

### 1.3 设计约束

- Hero 保持不动（用户确认 OK）
- 蓝色素材 `dialog-box.png` (1216×217) + `option-button.png` (900×60) 为可选增强，CSS 方案可完全替代
- 系统菜单无素材，全部 CSS 实现
- 新增蓝色色值已在 `globals.css` 的 `@theme` + `:root` 中声明

---

## 二、改动全景图

### 2.1 新建文件（4 个）

| # | 文件 | 组件名 | 用途 |
|---|------|--------|------|
| 1 | `src/components/DialogBox.tsx` | DialogBox | VN 对话框：白色圆角底框 + #2FA4D7 蓝底名字框(rounded-xl) + 蓝底文字框(rounded-md)，flex 比例 1:3 |
| 2 | `src/components/ChapterHeader.tsx` | ChapterHeader | VN 章节标题（"CHAPTER N"标签 + 标题 + 蓝色下划线） |
| 3 | `src/components/OptionButton.tsx` | OptionButton | VN 选项按钮（`#2FA4D7` 蓝底 + ▶ 箭头 + 白色文字，纯 CSS） |
| 4 | `src/components/SectionTransition.tsx` | SectionTransition | 过渡台词（`~ text ~` 居中 + 水平装饰线） |

### 2.2 修改文件（10 个）

| # | 文件 | 改动 |
|---|------|------|
| 1 | `src/styles/globals.css` | ✅ 已完成（新增 `--color-dialog-*` 三色值） |
| 2 | `src/lib/animations.ts` | ✅ 追加 5 个 VN variants（chapterReveal / dialogSlideUp / optionStagger / optionItem / transitionFade） |
| 3 | `src/messages/zh.json` | ✅ 已完成（新增 `transitions` 键） |
| 4 | `src/messages/ja.json` | ✅ 已完成 |
| 5 | `src/messages/en.json` | ✅ 已完成 |
| 6 | `src/components/HeroSection.tsx` | ✅ CTA 按钮替换为 `<OptionButton>`；按钮加高 `py-4`；间距 `mt-8`→`mt-6`，Scroll ↓ `mt-6`→`mt-4` |
| 7 | `src/components/CaseStudiesSection.tsx` | ✅ 顶部加 `<ChapterHeader chapterNumber={1} />` |
| 8 | `src/components/ProjectsSection.tsx` | ✅ 顶部加 `<ChapterHeader chapterNumber={2} />` |
| 9 | `src/components/AboutSection.tsx` | ✅ 顶部加 `<ChapterHeader chapterNumber={3} />`；bio 改为 `<DialogBox characterName="KONG JUN SIENG">` |
| 10 | `src/components/ContactSection.tsx` | ✅ 顶部加 `<ChapterHeader chapterNumber={4} subtitle="SELECT AN OPTION" />`；联系方式改为 `<OptionButton>` 列表 |
| 11 | `src/app/[lang]/page.tsx` | ✅ Section 之间插入 4 段 `<SectionTransition>` |

### 2.3 无需改动

| 文件 | 原因 |
|------|------|
| `src/app/[lang]/projects/[slug]/page.tsx` | 详情页路由、结构不变 |
| `src/components/Footer.tsx` | 结构不变 |
| `src/components/Navbar.tsx` | 结构不变（可后续加菜单装饰，非 P0） |
| `src/components/SkillTag.tsx` | 结构不变 |
| `src/components/SocialLink.tsx` | 结构不变 |

---

## 三、已就绪的基础设施

以下改动已在之前完成，无需重复操作：

| 项目 | 状态 |
|------|------|
| `--color-dialog-blue: #2FA4D7` 写入 `globals.css` `@theme` + `:root` | ✅ |
| `--color-dialog-dark: #1A8DB8` 同上 | ✅ |
| `--color-dialog-soft: #E5F4FC` 同上 | ✅ |
| `src/messages/{lang}.json` 追加 `transitions` 键 | ✅ |
| 蓝色素材分辨率已写入设计文档（`dialog-box.png` 1216×217, `option-button.png` 900×60） | ✅ |

---

## 四、执行任务表

### Phase 6：VN UI 组件构建（4 组件 + 基础设施）✅ 全部完成

| ID | 任务 | 优先级 | 依赖 | 工作量 | 验收标准 | 产出文件 | 状态 |
|----|------|--------|------|--------|---------|---------|:---:|
| VN-6.1 | **创建 `DialogBox.tsx`**：VN 对话框。三层圆角结构（白框 `rounded-lg` + 名字框 `rounded-xl` + 文字框 `rounded-md`）。底色 `#2FA4D7`，等比例 `aspect-[1200/225]`，内层 flex 1:3 固定比例。Props: `{ characterName?, children, className? }` | P0 | — | M | 进入视口对话框淡入上浮；名字框与文字框高度固定 1:3；文字白色；响应式自适应 | `DialogBox.tsx` | ✅ |
| VN-6.2 | **创建 `ChapterHeader.tsx`**：VN 章节标题。Props: `{ chapterNumber?: number, title, subtitle?, className? }`。渲染：`CHAPTER 01` 标签 (`text-dialog-blue`) → 标题 → 蓝色下划线装饰条 | P0 | — | M | 进入视口时：整体从左侧滑入（x:-30→0, 500ms） | `ChapterHeader.tsx` | ✅ |
| VN-6.3 | **创建 `OptionButton.tsx`**：VN 选项按钮。纯 CSS 单变体：蓝底白色文字，`▶` 指示符，`whileHover` 右移+放大。Props: `{ children, onClick?, className? }` | P0 | — | S | 按钮渲染为蓝底白字；hover 缩放+右移 4px；tap 缩回 | `OptionButton.tsx` | ✅ |
| VN-6.4 | **创建 `SectionTransition.tsx`**：过渡台词组件。居中显示 `~ text ~`，水平装饰线贯穿，`whileInView` 淡入 400ms。不使用 DialogBox。Props: `{ text, className? }` | P0 | — | M | 滚动到视口时，文字从中心淡入放大；`viewport: { once: true }` | `SectionTransition.tsx` | ✅ |
| VN-6.5 | **`animations.ts` 追加 5 个 VN variants**：`chapterReveal`（x:-30→0）、`dialogSlideUp`（y:24→0, scale:0.98→1）、`optionStagger` + `optionItem`、`transitionFade`（scale:0.92→1） | P0 | — | S | variants 在对应组件中使用 | `animations.ts` | ✅ |

### Phase 7：首页 Section 集成 VN 组件 ✅ 全部完成

| ID | 任务 | 优先级 | 依赖 | 工作量 | 验收标准 | 产出文件 | 状态 |
|----|------|--------|------|--------|---------|---------|:---:|
| VN-7.1 | **HeroSection 改造**：CTA 按钮替换为 `<OptionButton>`；按钮加高 `py-4`；button 上方 `mt-8`→`mt-6`，Scroll ↓ `mt-6`→`mt-4` | P0 | VN-6.3 | S | CTA 显示为蓝底白字按钮；间距合理 | `HeroSection.tsx` | ✅ |
| VN-7.2 | **CaseStudiesSection 改造**：顶部加 `<ChapterHeader chapterNumber={1} />`。CaseStudyCard 保持原始结构不变（描述不使用 DialogBox） | P0 | VN-6.2 | S | 章节标题显示 | `CaseStudiesSection.tsx` | ✅ |
| VN-7.3 | **ProjectsSection 改造**：顶部加 `<ChapterHeader chapterNumber={2} />` | P0 | VN-6.2 | S | 章节标题显示 | `ProjectsSection.tsx` | ✅ |
| VN-7.4 | **AboutSection 改造**：顶部加 `<ChapterHeader chapterNumber={3} />`。bio 替换为 `<DialogBox characterName="KONG JUN SIENG">`（叠在 PNG 位置，无独立外框） | P0 | VN-6.1, VN-6.2 | M | 章节标题显示；自我介绍在对话框内；名字显示在 PNG 上 | `AboutSection.tsx` | ✅ |
| VN-7.5 | **ContactSection 改造**：顶部加 `<ChapterHeader chapterNumber={4} subtitle="SELECT AN OPTION" />`。联系方式改为 `<OptionButton>` 列表（email/GitHub/LinkedIn/Resume），无菜单框架容器 | P0 | VN-6.2, VN-6.3 | S | 章节标题显示；四个选项按钮；stagger 入场 | `ContactSection.tsx` | ✅ |
| VN-7.6 | **Section 过渡台词集成**：`page.tsx` 中每个 Section 之间插入 `<SectionTransition>`，四段台词从 messages 读取 | P0 | VN-6.4 | M | 页面可见过渡线条 | `page.tsx` | ✅ |

### Phase 7b：验证 ✅ 完成

| ID | 任务 | 优先级 | 依赖 | 工作量 | 验收标准 | 状态 |
|----|------|--------|------|--------|---------|:---:|
| VN-7.7 | **lint + typecheck + build** | P0 | VN-7.1~7.6 | XS | `pnpm lint`、`pnpm typecheck`、`pnpm build` 全部通过 | ✅ |

---

## 五、验收清单（逐项对照检查）✅ 全部完成

### 视觉检查

- [x] Hero CTA 为蓝底白字选项按钮风格，hover 右移+放大
- [x] Case Studies 顶部显示 CHAPTER 01 — Case Studies（蓝色标签 + 下划线）
- [x] Case Study 卡片保持原始卡片结构（无 DialogBox）
- [x] Projects 顶部显示 CHAPTER 02 — 作品
- [x] About 顶部显示 CHAPTER 03 — About
- [x] About 自我介绍在对话框内，名字 "KONG JUN SIENG" 显示在蓝底名字框 (rounded-xl) 中
- [x] Contact 顶部显示 CHAPTER 04 — Contact + "SELECT AN OPTION" 副标题
- [x] Contact 联系方式为四个 OptionButton 列表（Email / GitHub / LinkedIn / Resume）
- [x] 简历下载按钮为 OptionButton 风格

### 交互检查

- [x] 章节标题进入视口时从左侧滑入 (x:-30→0)
- [x] 对话框进入视口时从底部淡入上浮 (y:24→0)
- [x] 选项按钮 hover 时放大 + 右移 4px
- [x] 选项按钮 tap 时有 scale(0.98) 反馈
- [x] Section 过渡台词在章节之间出现（`~ text ~` + 水平线）
- [x] About 对话框无逐字显示（未实现）

### 响应式检查

- [x] Mobile 无 hover 效果（选项按钮点击反馈仍在）
- [x] Mobile 过渡台词保留简化显示
- [x] Mobile 对话框无逐字效果（未实现）
- [x] 章节标题在 Mobile 正常显示（左侧滑入动画保留）

### 数据检查

- [x] `messages/{lang}.json` 的 `transitions` 键存在且内容正确
- [x] 三语台词正确匹配当前语言
- [x] `--color-dialog-*` CSS 变量在 Tailwind 中可用（`bg-dialog-blue`、`text-dialog-dark` 等）

---

## 六、回滚方案

如果 VN 改造出现问题需要回退：

| 层级 | 操作 |
|------|------|
| **组件级** | 删除 4 个新组件文件（DialogBox / ChapterHeader / OptionButton / SectionTransition） |
| **集成级** | 从各 Section 组件中移除 ChapterHeader / DialogBox / OptionButton 的引用，恢复原始 JSX |
| **数据级** | 从 messages JSON 中移除 `transitions` 键（无影响，其他功能不依赖） |
| **样式级** | `globals.css` 中 `--color-dialog-*` 可保留（未使用则无影响） |
