# 图片设计稿到可编辑 PPT 的复原文档

本文定义“图片设计稿 / 截图 / 多页 PDF / 图片版 PPT → 可编辑 PowerPoint”的处理方法。目标是让用户得到一个可以继续编辑文字、形状、图片资产和备注的 `.pptx`，而不是只有一张整页截图的 PPT。

## 1. 输入输出契约

### 输入

- 单张设计稿图片：输出 1 页 PPT。
- 多张图片：每张图片对应 1 页 PPT，按输入顺序排列。
- 多页 PDF：第 N 页对应 PPT 第 N 页。
- 图片版 PPT/PPTX：每页渲染成图片后重建，原备注按页复制。

### 输出

```text
output/<job-id>/
  deck_manifest.json
  pages/
    page_001/
      source.png
      layout.json
      manifest.json
      assets/
      page.pptx
      preview.png
      validation.json
  final/
    editable.pptx
    validation.json
```

最终交付：`final/editable.pptx`。

## 2. PPT 复原的核心策略

### 2.1 先拆层，再重建

1. **背景层**：纯色、渐变、规则纹理优先用 slide background 或 shape；复杂照片/插画用 clean base 图片。
2. **结构层**：网格、卡片、分栏、图表容器、时间线、箭头、装饰线用 PowerPoint shape。
3. **文本层**：标题、正文、标注、图表数值恢复为 text box。
4. **资产层**：logo、icon、插画、照片、复杂装饰拆为独立图片。
5. **备注层**：如果输入是 PPT/PPTX，备注原样复制，不交给视觉 worker 改写。

### 2.2 对象复原优先级

```text
原生文本 > 原生形状 > 原生图表/表格 > 独立透明图片 > 整页背景图
```

只有当对象复杂、低清、语义不确定或无法稳定编辑时，才退化为图片资产。

## 3. 页面分析步骤

### Step 1：画布与版式识别

- 读取源图宽高，确定 PPT 页面比例：16:9、4:3、A4 或自定义。
- 识别安全边距、主视觉区域、标题区、页脚、导航/章节标识。
- 建立坐标转换：`source px -> ppt point/EMU`。

### Step 2：文本 inventory

对每段文字记录：

```json
{
  "id": "txt_001",
  "text": "年度业绩概览",
  "bbox": [80, 96, 540, 64],
  "font_size": 42,
  "font_weight": 700,
  "color": "#111827",
  "align": "left",
  "line_height": 1.18,
  "confidence": 0.96
}
```

处理规则：

- 可读文字必须变成文本框。
- 文字过小或模糊时，保留视觉近似并标记 `needs_review`。
- 中英文混排要记录字体 fallback。
- 同一段落不要拆成过多文本框，避免用户编辑困难。

### Step 3：形状与图表重建

- 矩形、圆角矩形、圆形、线条、箭头、虚线、阴影用 PowerPoint shape。
- 表格优先用 PowerPoint table；如果样式复杂，可用线条 + 文本框。
- 图表若能读出数据，使用可编辑 chart；否则用形状拟合柱/线/饼，并把数据标记为估算。

### Step 4：复杂资产拆分

复杂资产包括：照片、人物、产品图、复杂插画、纹理背景、手绘风装饰等。

处理方式：

1. 从源图裁切候选区域。
2. 需要透明背景时进行抠图或 chroma-key asset sheet。
3. 对每个资产记录 `source_bbox`、hash、处理方式、置信度。
4. 在 PPT 中用图片对象插入，而不是嵌入整页截图。

### Step 5：生成页面 manifest

`manifest.json` 负责描述 PPT 构建需要的所有对象：

```json
{
  "slide": { "width": 13.333, "height": 7.5, "unit": "inch" },
  "objects": [
    { "type": "shape", "shape": "rect", "x": 0, "y": 0, "w": 13.333, "h": 7.5, "fill": "#F7F8FA" },
    { "type": "text", "text": "年度业绩概览", "x": 0.74, "y": 0.62, "w": 5.0, "h": 0.5 }
  ],
  "assets": []
}
```

## 4. 构建 PPTX

推荐使用确定性脚本执行构建：

1. 读取 `manifest.json`。
2. 创建空白 slide。
3. 先绘制背景，再绘制结构形状，再插入资产，最后放置文字。
4. 应用主题字体、颜色、母版页脚。
5. 复制备注。
6. 保存 `page.pptx`，最终合并为 `editable.pptx`。

## 5. QA 标准

| 维度 | 通过标准 |
| --- | --- |
| 页数 | 输出页数与输入页数一致 |
| 文字 | 关键标题、正文、数值无缺失，文本可选中编辑 |
| 位置 | 主要对象 bbox 与源图视觉偏差在可接受范围内 |
| 层级 | 遮挡、阴影、浮层顺序正确 |
| 资产 | 复杂图像独立存在，有 provenance |
| 可编辑性 | 不把整页截图作为唯一内容 |
| 备注 | PPT/PPTX 输入备注按页原样保留 |

## 6. 局部修复策略

- 文本错：只修对应文本框内容、字号或位置。
- 图标错：只重切或重生成该图标。
- 背景破：只修 clean base 或局部背景块。
- 层级错：只调整 z-index。
- 页面比例错：重新计算全页坐标，但不要改变已确认的文本内容。

## 7. 给傻瓜用户的 App 流程建议

1. 用户上传图片/PDF/PPT。
2. App 展示页面缩略图，并提示“先试 1 页”。
3. 后端生成 `layout.json` 和可编辑 PPT 预览。
4. 用户在 App 中选择“更像原图”或“更方便编辑”。
5. 对失败项执行局部修复。
6. 导出 `.pptx`，同时提供“可编辑对象清单”和“需要人工复核项”。
