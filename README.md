
# AI Skill Library · AI 技能库

> Two productivity skills for AI-powered travel planning and visual card generation.
> 两个 AI 技能，覆盖旅行行程规划与视觉卡片生成。

---

## Skills Overview · 技能一览

| Skill | Claude.ai | OpenClaw | 核心能力 |
|-------|-----------|----------|---------|
| 🗺️ Travel Planner | -| ✅ v2.3 | 多源搜索 · 每日行程 · 地图 |
| 🎨 Visual Design System | ✅ v2 | ✅ v2 | 6主题 · 9元素 · 配色推导 |

---

## 🗺️ Travel Planner

**一句话触发，自动生成完整旅行行程。**
Triggered by a single sentence, generates a complete day-by-day itinerary automatically.

### 平台差异 · Platform Differences

**Claude.ai 版 · Claude.ai Version**
- 交互式需求收集卡片 · Interactive requirement collection cards
- 内置地图生成（`places_search` + `places_map_display`）· Built-in interactive map generation
- Google Maps 实时餐厅 & 景点数据 · Real-time restaurant & attraction data
- 需要 Claude Pro · Requires Claude Pro

**OpenClaw 版 · OpenClaw Version**
- 通过 `openclaw browser` 驱动浏览器搜索 · Browser-driven search via `openclaw browser`
- OCR 自动检测：有 Tesseract/EasyOCR 时抓取小红书图片内容 · Auto OCR: extracts Xiaohongshu image content when Tesseract/EasyOCR is available
- 三种策略自适应切换（无OCR / 有OCR / 混合）· Three adaptive strategies (no OCR / OCR / hybrid)
- 行程可导出为本地文件 · Itinerary exportable as local file
- 无需 Claude Pro · No Claude Pro required

### 推荐用法 · Recommended Usage

```
触发示例 · Trigger examples:
"帮我规划 7 天清迈旅行，从北京出发"
"Plan a 7-day trip to Sri Lanka, departing from Shanghai"
```

---

## 🎨 Visual Design System

**让 AI 生成有设计感的 HTML 卡片、报告、幻灯片。**
Helps AI generate visually consistent HTML cards, reports, and slides.

### 核心特性 · Key Features

- **骨骼与皮肤分离 · Structure / Style Separation** — 预设9种结构元素独立于6套色彩主题，可根据用户需求主题、内容结构延展任意组合 · 9 structural elements decoupled from 6 color themes
- **6套预设主题 · 6 Preset Themes** — 净白 / 暖土 / 暗夜 / 云雾 / 深空 / 晨曦，基于色彩设计规范结构，可任意延展定义主题色
- **自定色推导 · Custom Color Derivation** — 提供一个主色，自动推导完整9色配色方案并验证对比度 · Provide one hex value, full 9-role palette derived automatically with contrast validation
- **强制设计规则 · Hard Design Rules** — CSS变量约束 · 最多三层级 · 8px间距栅格 · CSS variables only · max 3 type levels · 8px spacing grid
- **响应式 · Responsive** — 内置移动端断点覆盖 · Built-in mobile breakpoints

### 平台说明 · Platform Notes

两个版本文件内容相同，格式不同：
Both versions contain the same content, different formats:

- `.md` — 上传至 Claude.ai 对话直接使用 · Upload to Claude.ai conversation
- `.skill` — 放入 OpenClaw skills 目录加载 · Place in OpenClaw skills directory

### 推荐用法 · Recommended Usage

```
触发示例 · Trigger examples:
"帮我做一张知识卡片，用暗夜主题"
"Make a data dashboard card using Deep Space theme"
"用 #E86A2B 这个颜色做一张产品介绍卡片"
```

---

## 文件说明 · File Guide

| 文件 · File | 用途 · Purpose |
|------------|---------------|
| `travel-planner_skill_v2.3.md` | Travel Planner · Claude.ai & OpenClaw 通用 |
| `visual-design-system-v2-FULL.md` | VDS · Claude.ai 上传用 |
| `visual-design-systemv2.skill` | VDS · OpenClaw 加载用 |

---

*Author: Mirachen-cc · MIT License*
