---
name: visual-design-system
description: "A foundational visual design library for generating high-quality HTML artifacts — cards, slides, reports, dashboards. Use this skill whenever generating any visual HTML output, including info cards, knowledge cards, PPT-style slides, data reports, or any layout with cards and text. This skill ensures visual consistency, hierarchy, and design quality across all outputs. Other skills that produce HTML should explicitly reference this skill. Trigger when user asks to make cards, slides, reports, summaries, or any visually designed content."
---

# Visual Design System

一套保证视觉关系正确的设计基础库。核心原则：**用规则约束关系，用变量承载风格，用覆盖机制留个性化空间。**

**三条强制规则，任何情况不得违反：**
1. 所有样式值必须来自 CSS 变量，禁止直接写颜色值或随意 px 值
2. 每张卡片最多三个文字层级，强调元素最多一处
3. 所有间距必须是 `--space-base`（8px）的整数倍

---

## 使用流程

### Step 1：确定主题
- 用户未指定 → 默认「净白 Pure White」
- 用户指定预设名称 → 读取 `color/themes.md`，加载对应主题
- 用户提供自定色 → 读取 `color/matching.md`，推导完整 token

### Step 2：读取元素规范
读取 `structure/elements.md`，了解每种元素的定义和使用规则。

### Step 3：读取组合规则
读取 `composition/rules.md`，了解结构元素与色彩角色的绑定关系。

### Step 4：生成 HTML
在 `<style>` 顶部声明完整 CSS 变量，所有组件样式只引用变量。

---

## CSS 变量模板

```css
:root {
  --color-bg: ;
  --color-surface: ;
  --color-surface-alt: ;
  --color-surface-warm: ;
  --color-border: ;
  --color-border-light: ;
  --color-text-primary: ;
  --color-text-secondary: ;
  --color-text-caption: ;
  --color-accent: ;
  --color-accent-bg: ;

  --font-family: ;
  --font-size-heading: ;
  --font-size-subheading: ;
  --font-size-body: ;
  --font-size-caption: ;
  --font-weight-heavy: ;
  --font-weight-medium: ;
  --font-weight-light: ;
  --line-height-tight: ;
  --line-height-normal: ;

  --space-base: 8px;
  --space-xs:  8px;
  --space-sm:  16px;
  --space-md:  24px;
  --space-lg:  32px;
  --space-xl:  48px;

  --radius-sm: ;
  --radius-md: ;
  --radius-lg: ;
  --border-width: 1px;

  --shadow-none: none;
  --shadow-sm: ;
  --shadow-md: ;
}

@media (max-width: 768px) {
  :root {
    --space-base: 4px;
    --space-xs:  4px;
    --space-sm:  8px;
    --space-md:  16px;
    --space-lg:  24px;
    --space-xl:  32px;
  }
}
```

---

## 4点视觉质量自检

输出前验证，不满足则修正：
```
✓ 对比度   — 正文与背景 ≥ 4.5:1，标题 ≥ 3:1
✓ 字号梯度 — 标题是正文 1.5x 以上，辅助文字小于正文
✓ 间距一致 — 所有间距是 --space-base 的整数倍
✓ 强调克制 — 强调色面积 ≤ 20%，每卡片强调元素 ≤ 1 处
```

---

## 主题速查

| 编号 | 名称 | 适合场景 |
|------|------|---------|
| 01 | 净白 Pure White | 通用默认、中性内容 |
| 02 | 暖土 Warm Earth | 生活、消费、温暖感 |
| 03 | 暗夜 Dark Void | 科技、代码、深色 |
| 04 | 云雾 Cloud | 工具、文档、多分区 |
| 05 | 深空 Deep Space | 数据展示、产品对比 |
| 06 | 晨曦 Aurora | 情感、移动端、柔和 |

## 参考文件

| 文件 | 内容 | 何时读 |
|------|------|--------|
| `structure/elements.md` | 9种元素定义、规则、组合模式 | 每次必读 |
| `color/themes.md` | 6套预设主题完整 token 值 | 每次必读 |
| `color/matching.md` | 用户自定色的配色推导逻辑 | 用户指定颜色时读 |
| `composition/rules.md` | 元素+色彩的组合约束规则 | 每次必读 |

---

## 📐 结构层：元素规范

# 结构层：元素规范

每个元素定义四项：是什么 / 视觉特征 / 适合 / 不适合

---

## 01 主标题 Heading
**是什么：** 整张卡片最核心的信息，视觉优先级最高。
**视觉特征：** 字号 `--font-size-heading`，字重 `--font-weight-heavy`，颜色 `--color-text-primary`，行高 `--line-height-tight`
**适合：** 每张卡片只出现 1 次；不超过 2 行
**不适合：** 超过 2 行；使用强调色；加下划线或斜体

## 02 副标题 Subheading
**是什么：** 对主标题的补充，或多区块页面的二级标题。
**视觉特征：** 字号 `--font-size-subheading`，字重 `--font-weight-medium`，颜色 `--color-text-secondary`
**适合：** 紧跟主标题之下；每张卡片最多 2 次
**不适合：** 字号等于或大于主标题；颜色比主标题更深

## 03 正文 Body
**是什么：** 承载主要内容，可读性最重要。
**视觉特征：** 字号 `--font-size-body`，字重 `--font-weight-light`，颜色 `--color-text-primary`，行高 `--line-height-normal`
**适合：** 段落、列表、说明信息；每行 30-45 字（中文）
**不适合：** 使用强调色；超过 4 行不分段

## 04 标注 Caption
**是什么：** 最低层级的辅助信息，补充说明、来源、时间戳。
**视觉特征：** 字号 `--font-size-caption`，字重 `--font-weight-light`，颜色 `--color-text-caption`
**适合：** 图片说明；数据来源；时间标注；放在相关元素正下方
**不适合：** 比正文更显眼；承载核心信息；单独成块

## 05 标签 Tag / Badge
**是什么：** 分类标记或状态标识，快速传递属性。
**视觉特征：** 字号 `--font-size-caption`，字重 `--font-weight-medium`，背景 `--color-accent-bg`，文字 `--color-accent`，圆角 `--radius-sm`
**适合：** 分类（技术/设计）；状态（新/热门）；每张卡片不超过 3 个
**不适合：** 标签文字超过 5 个字；每张卡片超过 3 个

## 06 区块 Section
**是什么：** 将相关内容组织在一起的容器，页面基本组织单位。
**视觉特征：** 背景 `--color-surface`，圆角 `--radius-md`，内边距 `--space-md`，可选细边框 `--color-border`
**适合：** 将同类信息归组；嵌套时子区块用 `--color-surface-alt`
**不适合：** 单个区块超过 4 种元素类型；嵌套超过 2 层

## 07 分割线 Divider
**是什么：** 列表条目之间的轻量分割，不用于区块间分割。
**视觉特征：** 高度 1px，颜色 `--color-border`
**适合：** 列表条目间；区块内部分割
**不适合：** 区块与区块之间（用留白）；装饰用途；连续超过 5 条

## 08 数据指标 Metric
**是什么：** 突出展示关键数字，视觉冲击力最强的元素。
**视觉特征：** 数字字号为 `--font-size-heading` 的 1.5 倍，字重 `--font-weight-heavy`；单位说明用 `--font-size-caption` + `--color-text-caption`
**适合：** 每张卡片只有 1 个核心数字；数据卡片主体
**不适合：** 一张卡片超过 1 个等大数字；数字与说明字号相近

## 09 配图 Image / Illustration
**是什么：** 视觉辅助，增强内容理解或吸引力。
**视觉特征：** 圆角与所在区块一致；object-fit: cover；移动端优先满宽
**适合：** 产品图占卡片 50-70%；图标尺寸固定 32/48/64px
**不适合：** 图文重叠无蒙层；图片拉伸变形

---

## 元素组合模式

### Pattern A — 信息卡片
```
[标签] → [主标题] → [副标题] → [正文] → [标注]
```
适用：知识卡片、内容摘要、文章预览

### Pattern B — 数据卡片
```
[副标题/说明] → [数据指标（超大）] → [单位/标注]
```
适用：数据展示、指标对比、核心数字

### Pattern C — 功能入口卡片
```
[图标 32-48px] → [副标题] → [正文 1-2行]
```
适用：功能列表、导航网格、选项入口

### Pattern D — 图文卡片
```
左列: [配图 占50%] | 右列: [主标题] → [正文] → [标签]
```
适用：产品卡片、人物介绍、内容推荐

---

## 层级强制规则

```
每张卡片：
- 文字层级最多 3 层（标题 / 正文 / 标注）
- 强调色元素最多 1 处
- 标签最多 3 个
- 嵌套区块最多 2 层

整页：
- 每个区块有且只有 1 个主标题或核心数据指标
- 区块之间用留白分割，最小间距 --space-lg
- 禁止用线条分割区块，只用线条分割列表条目内部
```

---

## 🎨 色彩层：预设主题

# Color — Preset Themes

6 preset themes with complete token definitions.

---

## Theme 01 — 净白 Pure White
**风格**: 中性极简，通用性最强，默认主题
**适用场景**: 知识卡片、报告、任何需要中性专业感的内容

```css
--color-bg:             #FFFFFF;
--color-surface:        #F7F7F7;
--color-surface-alt:    #FAFAFA;
--color-surface-warm:   #FAFAFA;
--color-border:         #E5E5E5;
--color-border-light:   #EEEEEE;
--color-text-primary:   #111111;
--color-text-secondary: #888888;
--color-text-caption:   #AAAAAA;
--color-accent:         #111111;
--color-accent-bg:      #F0F0F0;
--font-family:          -apple-system, 'PingFang SC', 'Helvetica Neue', sans-serif;
--font-size-heading:    28px;
--font-size-subheading: 18px;
--font-size-body:       15px;
--font-size-caption:    12px;
--font-weight-heavy:    700;
--font-weight-medium:   500;
--font-weight-light:    400;
--line-height-tight:    1.2;
--line-height-normal:   1.6;
--radius-sm:  6px;
--radius-md:  10px;
--radius-lg:  16px;
--border-width: 1px;
--shadow-none: none;
--shadow-sm:   0 1px 4px rgba(0,0,0,0.06);
--shadow-md:   0 4px 24px rgba(0,0,0,0.10);
/* mobile: heading 22px, body 14px, radius-md 14px, radius-lg 20px */
```

---

## Theme 02 — 暖土 Warm Earth
**风格**: 暖灰白底，砖橙强调，自然有质感
**适用场景**: 产品介绍、生活方式内容、品牌展示

```css
--color-bg:             #F2F0EB;
--color-surface:        #FFFFFF;
--color-surface-alt:    #FAF9F6;
--color-surface-warm:   #FFF8F4;
--color-border:         #E5E2DC;
--color-border-light:   #F0EDE8;
--color-text-primary:   #1A1A1A;
--color-text-secondary: #888580;
--color-text-caption:   #AAA69F;
--color-accent:         #D4521A;
--color-accent-bg:      #FAE8DE;
--font-family:          Georgia, 'Songti SC', serif;
--font-size-heading:    28px;
--font-size-subheading: 16px;
--font-size-body:       14px;
--font-size-caption:    11px;
--font-weight-heavy:    800;
--font-weight-medium:   500;
--font-weight-light:    400;
--line-height-tight:    1.1;
--line-height-normal:   1.6;
--radius-sm:  4px;
--radius-md:  8px;
--radius-lg:  12px;
--border-width: 1px;
--shadow-none: none;
--shadow-sm:   0 1px 3px rgba(0,0,0,0.05);
--shadow-md:   0 4px 20px rgba(0,0,0,0.08);
/* mobile: heading 24px, body 14px, radius-md 12px, radius-lg 18px */
```

---

## Theme 03 — 暗夜 Dark Void
**风格**: 深炭灰底，冷紫强调，科技专业感
**适用场景**: 技术内容、代码展示、科技产品介绍

```css
--color-bg:             #1C1C1E;
--color-surface:        #2A2A2C;
--color-surface-alt:    #343436;
--color-surface-warm:   #343436;
--color-border:         #3A3A3C;
--color-border-light:   #444446;
--color-text-primary:   #F0F0F0;
--color-text-secondary: #888888;
--color-text-caption:   #666666;
--color-accent:         #7B5EA7;
--color-accent-bg:      #2D2440;
--font-family:          'SF Mono', 'Fira Code', -apple-system, 'PingFang SC', monospace;
--font-size-heading:    28px;
--font-size-subheading: 17px;
--font-size-body:       14px;
--font-size-caption:    12px;
--font-weight-heavy:    700;
--font-weight-medium:   500;
--font-weight-light:    400;
--line-height-tight:    1.2;
--line-height-normal:   1.6;
--radius-sm:  6px;
--radius-md:  12px;
--radius-lg:  18px;
--border-width: 1px;
--shadow-none: none;
--shadow-sm:   0 1px 4px rgba(0,0,0,0.3);
--shadow-md:   0 4px 24px rgba(0,0,0,0.5);
/* mobile: heading 22px, body 14px, radius-md 16px, radius-lg 24px */
```

---

## Theme 04 — 云雾 Cloud
**风格**: 白底，浅彩色区块点缀，轻盈多彩
**适用场景**: 工作台、知识库、分类展示、学习内容

```css
--color-bg:             #FFFFFF;
--color-surface:        #F5F5F0;
--color-surface-alt:    #FAFAF6;
--color-surface-warm:   #FFF8E7;
--color-border:         #E8E8E4;
--color-border-light:   #F0F0EC;
--color-text-primary:   #1A1A1A;
--color-text-secondary: #6B6B6B;
--color-text-caption:   #999999;
--color-accent:         #E8A020;
--color-accent-bg:      #FFF3DC;
--font-family:          -apple-system, 'PingFang SC', 'Helvetica Neue', sans-serif;
--font-size-heading:    24px;
--font-size-subheading: 16px;
--font-size-body:       14px;
--font-size-caption:    12px;
--font-weight-heavy:    600;
--font-weight-medium:   500;
--font-weight-light:    400;
--line-height-tight:    1.3;
--line-height-normal:   1.6;
--radius-sm:  8px;
--radius-md:  12px;
--radius-lg:  16px;
--border-width: 1px;
--shadow-none: none;
--shadow-sm:   0 1px 3px rgba(0,0,0,0.05);
--shadow-md:   0 2px 16px rgba(0,0,0,0.08);
/* mobile: heading 20px, body 14px, radius-md 16px, radius-lg 20px */
```

---

## Theme 05 — 深空 Deep Space
**风格**: 纯黑底，深灰卡片，极度克制，高级感强
**适用场景**: 产品发布、高端展示、数据对比卡片

```css
--color-bg:             #000000;
--color-surface:        #1C1C1E;
--color-surface-alt:    #2C2C2E;
--color-surface-warm:   #2C2C2E;
--color-border:         #2C2C2E;
--color-border-light:   #3A3A3C;
--color-text-primary:   #FFFFFF;
--color-text-secondary: #8E8E93;
--color-text-caption:   #636366;
--color-accent:         #0A84FF;
--color-accent-bg:      #001E3C;
--font-family:          -apple-system, 'PingFang SC', 'SF Pro Display', sans-serif;
--font-size-heading:    32px;
--font-size-subheading: 17px;
--font-size-body:       15px;
--font-size-caption:    12px;
--font-weight-heavy:    700;
--font-weight-medium:   500;
--font-weight-light:    400;
--line-height-tight:    1.1;
--line-height-normal:   1.5;
--radius-sm:  8px;
--radius-md:  16px;
--radius-lg:  20px;
--border-width: 0px;
--shadow-none: none;
--shadow-sm:   none;
--shadow-md:   0 8px 32px rgba(0,0,0,0.6);
/* mobile: heading 26px, body 15px, radius-md 20px, radius-lg 28px */
```

---

## Theme 06 — 晨曦 Aurora
**风格**: 粉紫渐变底，半透明白卡片，柔和圆润
**适用场景**: 情感类内容、移动端优先、生活记录、个人品牌

```css
--color-bg:             linear-gradient(135deg, #E8D5F0 0%, #F5C5C5 50%, #FDE8D8 100%);
--color-surface:        rgba(255,255,255,0.85);
--color-surface-alt:    rgba(255,255,255,0.65);
--color-surface-warm:   rgba(255,255,255,0.65);
--color-border:         rgba(255,255,255,0.6);
--color-border-light:   rgba(255,255,255,0.4);
--color-text-primary:   #1A1A1A;
--color-text-secondary: #9A8A8A;
--color-text-caption:   #BBB0B0;
--color-accent:         #E8748A;
--color-accent-bg:      rgba(232,116,138,0.12);
--font-family:          -apple-system, 'PingFang SC', 'Helvetica Neue', sans-serif;
--font-size-heading:    26px;
--font-size-subheading: 17px;
--font-size-body:       14px;
--font-size-caption:    12px;
--font-weight-heavy:    600;
--font-weight-medium:   500;
--font-weight-light:    400;
--line-height-tight:    1.3;
--line-height-normal:   1.6;
--radius-sm:  20px;
--radius-md:  20px;
--radius-lg:  28px;
--border-width: 1px;
--shadow-none: none;
--shadow-sm:   0 2px 12px rgba(200,150,180,0.15);
--shadow-md:   0 8px 32px rgba(200,150,180,0.25);
/* 注意：--color-bg 是渐变，应用于 body，不用于卡片 */
/* mobile: heading 22px, body 14px — 圆角已适配移动端 */
```

---

## Theme Selection Guide

| 用户描述 | 推荐主题 |
|---------|---------|
| 没有说 / 通用 | 净白 Pure White |
| 简洁、专业、干净 | 净白 Pure White |
| 暖色、自然、有质感 | 暖土 Warm Earth |
| 科技、代码、深色 | 暗夜 Dark Void |
| 轻盈、多彩、工作台 | 云雾 Cloud |
| 高端、黑色、发布会 | 深空 Deep Space |
| 粉色、柔和、移动端 | 晨曦 Aurora |

---

## 🔬 色彩层：自定色配色推导

# Color — 自定色配色推导

当用户提供主色时，按以下逻辑推导完整 token。

---

## 推导步骤

### Step 1：判断主色属性
- 冷暖：红/橙/黄 → 暖；绿/蓝/紫 → 冷
- 明暗：亮度 > 60% → 浅色；< 40% → 深色
- 饱和度：> 70% → 鲜艳；< 30% → 低调

### Step 2：决定背景方向

| 主色类型 | 背景方向 |
|---------|---------|
| 鲜艳深色（深蓝、深绿） | 浅中性背景（白/浅灰） |
| 鲜艳浅色（浅橙、浅粉） | 白色背景，主色做强调 |
| 低饱和暖色（砖红、棕） | 暖灰背景（微黄/微橙底） |
| 低饱和冷色（灰蓝、灰绿） | 冷白背景（微蓝/微绿底） |
| 用户明确要深色 | 深灰 #111111 或纯黑 #000000 |

### Step 3：按角色推导

```
强调色：      = 用户提供的主色
背景色：      主色加白至 95% 亮度，饱和度降至 5% 以内
表面色：      #FFFFFF（浅色主题）
次级表面：    背景色加白 3%
次级表面暖：  背景色加白 5% + 微暖调
边框色：      背景色加深 8%
轻边框色：    背景色加深 5%
主文字：      #111111（浅色）/ #F0F0F0（深色）
辅助文字：    #555555（浅色）/ #888888（深色）
标注文字：    #999999（浅色）/ #666666（深色）
强调色背景：  主色加白至 92%，保留 12% 饱和度
```

### Step 4：对比度验证

```
主文字 vs 背景色    → ≥ 7:1（必须）
辅助文字 vs 表面色  → ≥ 4.5:1（必须）
强调色 vs 背景色    → ≥ 3:1（必须）
```

不满足时：加深文字色或降低强调色亮度，不改背景色。

---

## 推导示例

### `#1A6FE8`（科技蓝）
```css
--color-bg:             #F5F8FF;
--color-surface:        #FFFFFF;
--color-surface-alt:    #EEF3FD;
--color-surface-warm:   #F5F8FF;
--color-border:         #D6E4F7;
--color-border-light:   #E5EEFB;
--color-text-primary:   #111111;
--color-text-secondary: #555555;
--color-text-caption:   #999999;
--color-accent:         #1A6FE8;
--color-accent-bg:      #EBF2FF;
```

### `#16A34A`（清新绿）
```css
--color-bg:             #F5FAF7;
--color-surface:        #FFFFFF;
--color-surface-alt:    #F0F7F3;
--color-surface-warm:   #F5FAF7;
--color-border:         #D1EAD9;
--color-border-light:   #E0F0E6;
--color-text-primary:   #111111;
--color-text-secondary: #555555;
--color-text-caption:   #999999;
--color-accent:         #16A34A;
--color-accent-bg:      #DCFCE7;
```

### `#F26419`（亮橙）
```css
--color-bg:             #FDF6F0;
--color-surface:        #FFFFFF;
--color-surface-alt:    #FEFAF7;
--color-surface-warm:   #FFF5EE;
--color-border:         #F0E4D8;
--color-border-light:   #F5EDE4;
--color-text-primary:   #111111;
--color-text-secondary: #888888;
--color-text-caption:   #AAAAAA;
--color-accent:         #F26419;
--color-accent-bg:      #FEF0E6;
```

---

## 色彩搭配相性

| 主色 | 适合搭配 | 避免 |
|-----|---------|------|
| 蓝色系 | 浅橙/金黄或同系浅蓝 | 绿色（层次不足） |
| 绿色系 | 浅棕/米白或珊瑚红 | 红色（对比过强） |
| 红/橙系 | 深棕/米白/中性灰 | 绿色（圣诞感） |
| 紫色系 | 浅金/白/冷灰 | 橙色（刺眼） |
| 中性灰 | 任一饱和度适中的色 | 多色强调（失去克制） |

---

## 🔗 组合规则：结构与色彩的绑定

# 组合规则：结构与色彩绑定

---

## 元素与色彩角色绑定表

| 元素 | 背景 | 文字 | 边框 |
|------|------|------|------|
| 页面背景 | `--color-bg` | — | — |
| 卡片 Card | `--color-surface` | — | `--color-border`（可选）|
| 嵌套区块（信息型）| `--color-surface-alt` | — | `--color-border-light` |
| 嵌套区块（强调型）| `--color-surface-warm` | — | `--color-border-light` |
| 主标题 | — | `--color-text-primary` | — |
| 副标题 | — | `--color-text-secondary` | — |
| 正文 | — | `--color-text-primary` | — |
| 标注 | — | `--color-text-caption` | — |
| 标签 Tag | `--color-accent-bg` | `--color-accent` | — |
| 分割线 | — | — | `--color-border`（1px）|
| 数据指标 | — | `--color-text-primary` 或 `--color-accent` | — |
| 主按钮 | `--color-accent` | `#FFFFFF` | — |
| 次按钮 | transparent | `--color-accent` | `--color-accent` |

---

## 禁止的组合

```
✗ 直接写颜色值（必须用变量）
✗ 直接写 px 字号或间距（必须用变量）
✗ 同一卡片超过 2 处使用 --color-accent
✗ 标题使用 --color-accent
✗ 区块间用线条分割（用留白代替）
✗ 卡片嵌套超过 2 层
✗ 阴影用于常规卡片（只用于浮层/弹窗）
```

---

## 典型卡片组合模式

### 模式 A：信息卡片
```
卡片背景：    --color-surface
├── 标签：    --color-accent-bg + --color-accent 文字
├── 主标题：  --color-text-primary，heading，heavy
├── 正文：    --color-text-primary，body，light
└── 标注：    --color-text-caption，caption
```

### 模式 B：数据指标卡片
```
卡片背景：    --color-surface
├── 标注（指标名）：--color-text-caption，caption
├── 数字（核心值）：--color-text-primary，heading×1.5，heavy
└── 说明（单位）：  --color-text-secondary，caption
```

### 模式 C：功能入口卡片（网格）
```
卡片背景：    --color-surface-alt
├── 图标：    --color-accent，32px
├── 标题：    --color-text-primary，subheading，medium
└── 说明：    --color-text-secondary，body
```

### 模式 D：步骤列表
```
每条步骤：    --color-surface-alt + --color-border-light 边框
├── 序号圆点：--color-accent 描边，caption，heavy
├── 标题：    --color-text-primary，body，medium
└── 说明：    --color-text-secondary，caption
```

---

## 间距组合规则

```
页面外边距：  --space-lg
卡片内边距：  --space-md
卡片间距：    --space-sm
元素间距：    --space-sm（异类）/ --space-xs（同类）
标签行间距：  --space-xs
图标与文字：  --space-xs
```

---

## 响应式调整

移动端（≤768px）色彩角色不变，调整：
```
卡片内边距：  --space-md → --space-sm
网格列数：    自动折叠为 1-2 列
图文卡片：    左右布局 → 上下布局
圆角：        通过断点自动切换（已在变量模板定义）
```