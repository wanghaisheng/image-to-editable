# 图片效果图到可点击互动 HTML Demo 的复原文档

本文把“图片复原为可编辑 PPT”的方法应用到 App 设计稿：从一张或多张效果图图片复原为可点击、可预览、可继续开发的 HTML/CSS/TypeScript Demo。目标不是只做像素级截图复刻，而是生成可维护的页面结构、组件、样式 token 和交互热点。

## 1. 目标产物

```text
html-demo/
  index.html
  src/
    app.ts
    router.ts
    interactions.ts
    screens/
      home.screen.ts
      detail.screen.ts
    styles/
      tokens.css
      layout.css
      components.css
  assets/
    logo.png
    icon-search.svg
    hero-card.png
  layout/
    screen_home.layout.json
    screen_detail.layout.json
  qa/
    preview_home.png
    validation_home.json
```

输出必须满足：

- 页面视觉接近输入效果图。
- 文本、按钮、列表、Tab、导航、弹窗等成为 DOM 元素或组件。
- 可点击区域有明确事件：跳转、打开弹窗、切换 Tab、展开折叠、表单输入等。
- 复杂视觉资产作为独立图片，不整屏贴图。
- Layout JSON 可被重新生成 HTML，也可迁移到 Capacitor、React、Vue 或游戏 UI。

## 2. App 效果图的对象模型

| 效果图对象 | HTML 复原类型 | 交互含义 |
| --- | --- | --- |
| 状态栏、导航栏 | `header`, `.status-bar`, `.nav-bar` | 返回、菜单、标题 |
| Tab Bar | `nav[role=tablist]` | 切换页面或状态 |
| 按钮 | `button` | click handler |
| 卡片 | `article`, `.card` | 可点击详情或静态展示 |
| 列表 | `ul/li`, virtual list | item click、滑动 |
| 输入框 | `input`, `textarea` | focus、change、submit |
| 弹窗/抽屉 | `dialog`, overlay | open/close |
| 图标 | SVG / icon font / PNG | 装饰或按钮内图标 |
| 图片 | `img`, CSS background | 视觉资产 |
| 图表 | SVG/canvas/HTML chart | tooltip/filter |

## 3. Screen Layout JSON

```json
{
  "schema_version": "1.0",
  "screen": {
    "id": "home",
    "name": "首页",
    "source": "input/home.png",
    "width": 390,
    "height": 844,
    "device": "iphone_15",
    "density": 3
  },
  "tokens": {
    "colors": {
      "bg": "#F7F8FA",
      "primary": "#2563EB",
      "text": "#111827",
      "muted": "#6B7280"
    },
    "radii": { "card": 16, "button": 12 },
    "spacing": { "xs": 4, "sm": 8, "md": 16, "lg": 24 }
  },
  "components": [
    {
      "id": "btn_start",
      "type": "button",
      "bbox": [24, 720, 342, 52],
      "z": 40,
      "text": "开始体验",
      "style": { "background": "#2563EB", "color": "#FFFFFF", "radius": 14 },
      "interaction": { "event": "click", "action": "navigate", "target": "detail" },
      "confidence": 0.95
    }
  ],
  "assets": [
    {
      "id": "hero_illustration",
      "type": "image",
      "path": "assets/hero_illustration.png",
      "bbox": [32, 160, 326, 220],
      "fit": "contain"
    }
  ]
}
```

## 4. 复原流程

### Step 1：屏幕归一化

- 将每张效果图识别为一个 screen。
- 记录设备尺寸、像素密度、安全区、状态栏高度、底部 home indicator。
- 多张图时，按文件名、用户标注或视觉相似性建立路由关系。

### Step 2：组件识别

按优先级识别：

1. 可交互控件：按钮、Tab、输入框、开关、列表项、返回按钮。
2. 页面结构：导航、标题区、卡片栅格、内容列表、底栏。
3. 文本与数据：标题、价格、状态、日期、标签。
4. 图片资产：头像、封面、插画、图标。
5. 装饰效果：阴影、渐变、模糊、分割线。

### Step 3：从绝对定位到可维护布局

初稿可以使用绝对定位保证相似度：

```css
.screen-home {
  position: relative;
  width: 390px;
  min-height: 844px;
}
.btn-start {
  position: absolute;
  left: 24px;
  top: 720px;
  width: 342px;
  height: 52px;
}
```

通过二次归纳，把明显结构转换为弹性布局：

- 垂直列表：`display: flex; flex-direction: column; gap: ...`。
- 卡片网格：`display: grid`。
- 底部 Tab：`position: fixed; bottom: env(safe-area-inset-bottom)`。
- 响应式容器：`max-width: 430px; margin: 0 auto`。

### Step 4：生成交互

交互从三个来源推断：

1. 视觉语义：按钮、Tab、返回箭头、关闭图标默认可点击。
2. 多屏关系：相同元素在不同截图中的状态变化表示切换或跳转。
3. 用户标注：用户可在 App 中点选区域并指定行为。

交互 JSON：

```json
{
  "interactions": [
    { "id": "i_start", "trigger": "#btn_start", "event": "click", "action": "navigate", "target": "detail" },
    { "id": "i_like", "trigger": "#btn_like", "event": "click", "action": "toggle_class", "class": "is-active" },
    { "id": "i_modal", "trigger": "#btn_filter", "event": "click", "action": "open_dialog", "target": "filter_dialog" }
  ]
}
```

TypeScript 绑定：

```ts
export function bindInteractions(root: HTMLElement) {
  root.querySelector('#btn_start')?.addEventListener('click', () => navigateTo('detail'));
  root.querySelector('#btn_like')?.addEventListener('click', (event) => {
    (event.currentTarget as HTMLElement).classList.toggle('is-active');
  });
}
```

## 5. 视觉资产处理

- Icon 优先替换为 SVG，保证清晰和可改色。
- 头像、照片、插画保留为图片资产。
- 阴影和圆角用 CSS，还原不了的复杂光效才使用 PNG。
- 背景纹理可拆为 `background-image`，但文本必须是 DOM。

## 6. QA 与验收

| 检查项 | 方法 |
| --- | --- |
| 视觉相似度 | 浏览器截图与源图对比 |
| 文本完整性 | Layout JSON 文本清单与 DOM 文本比对 |
| 点击可用性 | Playwright/单元测试触发 click |
| 响应式 | 典型宽度 360/390/430/768 px 截图 |
| 可维护性 | 禁止整屏唯一背景图，关键控件必须有 DOM |
| 无障碍 | button/input 使用语义标签和 aria-label |

## 7. 适配 Capacitor App

建议把 HTML Demo 当作 Capacitor App 的第一阶段产物：

1. `layout.json` 生成 Web Demo。
2. 用户在手机 WebView 中预览与点击。
3. 用户通过可视化标注修正交互。
4. 稳定后迁移到目标框架：React/Vue/Svelte/Ionic。
5. 原始 `layout.json` 继续作为设计回归测试基线。

## 8. 最小可行版本

MVP 可以先做：

- 单屏图片 → 单 HTML 页面。
- 文本框、按钮、卡片、图片四类对象。
- 绝对定位 CSS。
- 用户手动标注点击跳转。
- 浏览器截图 QA。

第二阶段再做：多屏自动路由、响应式布局归纳、组件库映射、状态机和表单交互。
