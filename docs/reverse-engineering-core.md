# 图片复原为可编辑产物的核心流程与逻辑

本文基于仓库中的 `image-to-editable-ppt-skill-main.zip` 与 `skill_duo_intro.pdf` 整理。ZIP 展示了一套面向 Codex 的“图片 / PDF / 图片版 PPT → 可编辑 PPTX”Skill；PDF 是 20 页栅格化介绍稿，强调两个互补技能：`codex-ppt` 负责“从内容生成视觉 PPT”，`image-to-editable-ppt` 负责“从视觉页面重建可编辑 PPT”。

## 1. 目标与关键思想

图片复原不是“把原图贴进容器”，而是把一张效果图拆成可编辑对象：

1. **文字层**：可读文本恢复为原生文本框，保留字号、字重、颜色、对齐、行距与层级。
2. **形状层**：规则几何、卡片、表格、图表、分割线、按钮等恢复为宿主平台原生形状或组件。
3. **图片资产层**：复杂插画、照片、纹理、图标、阴影、不可稳定矢量化的视觉元素保留为独立图片资产，并记录来源与裁切方式。
4. **交互 / 绑定层**：当目标不是 PPT，而是 HTML 或游戏引擎 Prefab 时，需要额外把可点击区域、组件状态、数据字段、脚本绑定写入中间描述。
5. **审计层**：每个对象都应有来源、置信度、复原策略、已知限制与 QA 证据。

这套方法的核心产物不是单一文件，而是“**中间 Layout JSON + 平台构建器 + 可视化验证 + 局部修复循环**”。

## 2. 端到端流水线

```text
输入文件
  ├─ 单张/多张图片
  ├─ 多页 PDF
  └─ 图片版 PPT/PPTX
        │
        ▼
输入归一化
  ├─ 每页渲染为 source.png
  ├─ 保留页序、画布尺寸、DPI、备注/元数据
  └─ 建立 run/page 目录和 page_request
        │
        ▼
逐页视觉理解
  ├─ 识别文本、形状、图片、图表、控件、状态
  ├─ 建立 reading order / z-index / bbox
  └─ 判定每个元素的复原策略
        │
        ▼
资产拆分与背景修复
  ├─ 简单背景用原生样式重建
  ├─ 复杂背景生成 clean base
  ├─ 复杂前景拆成透明资产
  └─ 记录 asset provenance
        │
        ▼
生成中间 Layout JSON
  ├─ canvas / theme / assets
  ├─ elements / components / constraints
  ├─ interactions / bindings / states
  └─ qa / known_limits
        │
        ▼
目标平台构建
  ├─ PPTX：python-pptx / OpenXML / 模板母版
  ├─ HTML：DOM + CSS + JS/TS 路由与事件
  └─ Prefab：节点树 + 组件 + 资源引用 + 绑定脚本
        │
        ▼
预览、对比、验证、修复
  ├─ 渲染 preview
  ├─ 与 source 做视觉差异检查
  ├─ 检查文本/资产/结构完整性
  └─ 只对失败对象做最小返工
```

## 3. 页面对象分类与处理策略

| 对象类型 | 识别依据 | 优先复原方式 | 退化策略 |
| --- | --- | --- | --- |
| 标题、正文、标签、数值 | OCR/视觉模型可读，边界清晰 | 原生文本对象 | 不确定字符标注 `needs_review` |
| 纯色/渐变背景 | 规则填充、低纹理 | CSS/PPT shape/引擎 Sprite 纯色或渐变 | 背景图 |
| 卡片、按钮、输入框 | 矩形、圆角、边框、阴影 | 原生形状/组件 | 图片切片 + 透明点击层 |
| 表格 | 网格线、单元格文本 | 原生表格或 flex/grid | 形状线 + 文本框 |
| 图表 | 坐标轴、柱/线/饼 | 可编辑 chart 或形状重建 | 静态图表图 + 数据占位 |
| 图标 | 小尺寸符号、常用语义 | SVG/icon font/平台图标 | 透明 PNG 资产 |
| 照片/插画/复杂纹理 | 高纹理、语义图像 | 独立图片资产 | clean base + 前景图层 |
| 手机 App 控件 | 导航栏、Tab、按钮、卡片、列表项 | HTML/组件树/可点击区域 | 静态图 + hotspot |
| 游戏 UI | Canvas、Panel、Button、Label、Sprite | Prefab 节点和组件 | Sprite 切片 + 绑定脚本 |

## 4. 中间 Layout JSON 的统一抽象

建议所有目标平台先生成统一 JSON，再由不同构建器转换为 PPT、HTML 或 Prefab。

```json
{
  "schema_version": "1.0",
  "source": {
    "path": "input/page_001.png",
    "width": 1440,
    "height": 900,
    "dpi": 144,
    "page_index": 1
  },
  "canvas": {
    "width": 1440,
    "height": 900,
    "background": { "type": "color", "value": "#F7F8FA" }
  },
  "theme": {
    "fonts": [{ "role": "sans", "family": "Inter" }],
    "colors": { "primary": "#2F6BFF", "text": "#1F2937" }
  },
  "assets": [
    {
      "id": "asset_logo",
      "type": "image",
      "path": "assets/logo.png",
      "source_bbox": [64, 40, 128, 72],
      "provenance": "cropped_from_source",
      "confidence": 0.93
    }
  ],
  "elements": [
    {
      "id": "title_hero",
      "type": "text",
      "bbox": [80, 96, 760, 156],
      "z": 10,
      "text": "年度业绩概览",
      "style": { "font_size": 42, "font_weight": 700, "color": "#111827" },
      "editable": true,
      "confidence": 0.96
    }
  ],
  "interactions": [],
  "bindings": [],
  "qa": {
    "known_limits": [],
    "needs_human_review": []
  }
}
```

### 坐标规范

- 坐标统一使用源图像素坐标：`[x, y, width, height]`。
- 输出到 PPT 时换算为 EMU / point；输出到 HTML 时换算为 CSS px、rem 或百分比；输出到游戏引擎时换算为 anchor、position、sizeDelta。
- 每个元素必须记录 `z`，避免阴影、遮罩、浮层、弹窗顺序错误。
- 需要响应式的 HTML/App 可以在 JSON 中增加 `constraints`：`pin`, `hug`, `fill`, `aspect_ratio`, `stack_group`。

## 5. 质量验证与修复闭环

每页/每屏至少输出：

1. `layout.json`：结构化对象清单。
2. `preview.png`：由目标产物重新渲染得到的预览。
3. `asset_contact_sheet.png`：全部切分资产的总览。
4. `validation.json`：文本、资产、层级、尺寸、视觉差异检查结果。
5. `repair_queue.json`：只包含需要局部修复的对象。

修复原则：

- **优先局部修复**：缺一个图标就修一个图标，不重建整页。
- **优先提高可编辑性**：能用原生文本/形状/组件就不要截图。
- **允许混合结果**：复杂视觉可用图片资产，但文字、按钮、布局应尽量可编辑。
- **明确不可恢复项**：低清文字、被遮挡内容、语义不确定图标必须标注置信度和人工复核点。

## 6. 可复用的软件架构

```text
packages/
  vision-analyzer/       # 图片理解、OCR、元素分类、bbox、阅读顺序
  layout-schema/         # Layout JSON schema、校验、版本迁移
  asset-pipeline/        # crop、透明化、clean base、asset sheet、hash
  ppt-builder/           # Layout JSON -> PPTX
  html-builder/          # Layout JSON -> HTML/CSS/TS
  prefab-builder/        # Layout JSON -> Prefab + binding TS
  qa-renderer/           # 目标产物渲染、截图、视觉 diff
  repair-orchestrator/   # 失败对象重试、局部修复、状态机
```

主流程负责调度和状态推进；页面/屏幕 worker 负责单页理解与构建；构建器必须尽量确定性，避免同一个 `layout.json` 每次输出不同结果。
