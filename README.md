# image-to-editable

本仓库用于整理“从图片复原为可编辑产物”的核心逻辑。当前分析对象包括：

- `image-to-editable-ppt-skill-main.zip`：一套图片 / PDF / 图片版 PPT 到可编辑 PPTX 的 Skill 实现。
- `skill_duo_intro.pdf`：20 页介绍稿，说明 `codex-ppt` 与 `image-to-editable-ppt` 的分工：前者从内容生成视觉 PPT，后者从视觉页面重建可编辑 PPT。

## 输出文档

- [图片复原为可编辑产物的核心流程与逻辑](docs/reverse-engineering-core.md)
- [图片设计稿到可编辑 PPT 的复原文档](docs/image-design-to-editable-ppt.md)
- [图片效果图到可点击互动 HTML Demo 的复原文档](docs/image-screenshot-to-interactive-html.md)
- [图片效果图到游戏引擎 Prefab 的复原文档](docs/image-screenshot-to-game-prefab.md)


## Image-first + 可编辑重建方法

本仓库现在把 PPT 场景明确拆成两个阶段：

1. **image2 / imagegen 视觉定稿**：先生成每页独立的 `slideXX_reference.png`，用真实图片锁定版式、配色、复杂插图、图标、地图和图表卡片效果。
2. **PowerPoint / Presentations 可编辑重建**：不要把整页图贴进 PPT，而是裁取局部复杂视觉资产，并用 PPT 原生文本框、形状、表格、箭头、卡片、标签和结论条重建可编辑结构。

关键红线：最终 PPT 不能是一张整页截图；可读文字、卡片、箭头、标签、结论条应尽量可编辑；只有复杂插画、地图、图标组、纹理和难以可靠重画的图表卡片才作为局部截图资产保留。详细规则见 [图片设计稿到可编辑 PPT 的复原文档](docs/image-design-to-editable-ppt.md)。

## 核心结论

复原流程不应直接把整张图嵌入目标产物，而应先把图片拆解为文本、形状、图片资产、交互/绑定与 QA 证据，再通过中间 `layout.json` 转换为不同平台产物：

```text
图片 / PDF / 图片版 PPT
  -> 页面归一化
  -> 视觉理解与对象分层
  -> 资产拆分与背景修复
  -> layout.json
  -> PPTX / HTML Demo / 游戏 Prefab
  -> 预览对比与局部修复
```

该方法可服务于目标产品：帮助普通用户从截图、设计稿或现有页面快速生成精美、可编辑、可互动的 PPT / App Demo / 游戏 UI 原型。
