# Convex + ChatGPT 驱动的效果图到可编辑产物产品计划

本文在现有“图片复原为可编辑产物”方法之上，定义一个面向三类创作者的产品计划：**PPT 创作者、App UI 创作者、Web/PC 设计创作者**。产品目标是让用户从一个点子出发，通过多轮对话确认内容大纲、信息架构、视觉风格和页面清单，先生成高保真效果图，再转换成可编辑、可交互、可继续开发的目标产物。

## 1. 产品定位

### 1.1 一句话定位

面向非专业设计师和轻量产品团队的 AI 设计生产台：从想法到效果图，再到可编辑 PPT、可点击 App Demo、可开发 Web/React 页面或游戏 UI Prefab。

### 1.2 核心承诺

- **从点子开始**：用户不需要先准备完整文档或设计稿，可以用自然语言描述需求。
- **多轮确认**：AI 不直接一次性生成最终文件，而是先帮助用户确认目标、受众、内容大纲、页面结构、风格和交互。
- **先视觉定稿**：通过图像模型生成每页/每屏效果图，让用户先确认“看起来对不对”。
- **再可编辑重建**：基于确认后的效果图，拆解为文本、形状、组件、图片资产、交互和绑定。
- **跨产物输出**：同一套 `layout.json` 可生成 PPTX、HTML、React 组件、App Demo、Web PC 页面或游戏引擎 Prefab。

### 1.3 不承诺的边界

- 不承诺任意截图都能 100% 像素级且 100% 原生对象复原。
- 不把整页/整屏效果图直接作为唯一背景交付。
- 不把低清、模糊、不可读文字自动当作确定内容，应进入人工确认队列。
- 不把复杂插画、照片、纹理、密集图表强行转换成可编辑矢量对象，必要时应保留为局部图片资产。

## 2. 目标用户与三条主旅程

### 2.1 PPT 创作者旅程

### 用户画像

- 咨询顾问、销售、运营、研究员、学生、创业者。
- 有主题、材料或报告，但缺少设计能力。
- 目标是快速得到美观且可继续编辑的 PPT。

### 旅程步骤

```text
点子 / 文档 / 资料
  -> 多轮对话确认汇报目标、受众、页数、结构
  -> 生成 PPT 大纲和逐页讲述逻辑
  -> 确认视觉风格、品牌色、版式密度
  -> 生成每页 slideXX_reference.png 效果图
  -> 用户逐页批注修改
  -> 图像理解与对象分层
  -> 转换为 editable.pptx
  -> 渲染预览与参考图对比
  -> 局部修复后交付
```

### 关键确认点

- 汇报对象：老板、客户、投资人、评委、课堂。
- PPT 类型：商业计划书、项目汇报、数据复盘、论文答辩、产品介绍。
- 页数与章节结构。
- 每页信息密度：极简、标准、信息密集。
- 风格：科技、咨询、学术、温暖、国企、互联网、奢华、儿童等。
- 可编辑程度要求：全部文字可编辑、图表可编辑、复杂视觉局部图片保留。

### 交付物

```text
ppt-output/
  deck_manifest.json
  outline.md
  style_guide.json
  pages/page_001/source.png
  pages/page_001/layout.json
  pages/page_001/assets/
  final/editable.pptx
  final/preview_contact_sheet.png
  final/validation.json
```

### 2.2 App UI 创作者旅程

### 用户画像

- 创业者、产品经理、独立开发者、UI 初学者。
- 有 App 点子，需要快速看到手机端原型。
- 目标是得到可点击 Demo，后续可迁移到 React Native、Flutter、Ionic 或原生开发。

### 旅程步骤

```text
App 点子
  -> 多轮对话确认目标用户、核心场景、MVP 功能
  -> 生成信息架构和关键用户流
  -> 确认设备尺寸、设计风格、组件规范
  -> 生成首页、详情页、表单页、弹窗、状态页效果图
  -> 用户标注按钮跳转和状态变化
  -> 转换为 HTML/CSS/TypeScript 可点击 Demo
  -> 可选导出 React 组件或移动端 WebView 项目
  -> Playwright 点击测试和截图回归
```

### 关键确认点

- App 类型：工具、社交、电商、教育、健康、内容、AI 助手。
- 用户任务：首次进入、搜索、下单、发布、收藏、支付、设置。
- 设备：iPhone、Android、平板或自定义尺寸。
- 页面清单：登录、首页、详情、列表、搜索、个人中心、设置、弹窗。
- 交互：跳转、Tab 切换、打开弹窗、表单输入、开关、筛选、展开折叠。
- 状态：空状态、加载、错误、成功、会员态、未登录态。

### 交付物

```text
app-demo/
  product_brief.md
  user_flows.json
  style_tokens.json
  screens/home/reference.png
  screens/home/layout.json
  src/index.html
  src/app.ts
  src/router.ts
  src/interactions.ts
  src/styles/tokens.css
  assets/
  qa/playwright-report/
```

### 2.3 Web/PC 设计创作者旅程

### 用户画像

- SaaS 创业者、B2B 产品经理、前端开发者、运营落地页负责人。
- 需要快速生成官网、后台、仪表盘、营销页或 Web App。
- 目标是得到可开发的 HTML/React 页面，而不是静态图片。

### 旅程步骤

```text
Web/PC 点子
  -> 多轮对话确认业务目标、页面类型、信息架构
  -> 生成站点地图、页面区块和关键 CTA
  -> 确认桌面宽度、响应式断点、设计系统
  -> 生成 PC 页面效果图和必要移动端变体
  -> 转换为 HTML/CSS/TypeScript 或 React 组件
  -> 抽取 tokens、组件、布局约束和交互
  -> 截图回归、响应式检查、可访问性检查
  -> 交付可继续开发项目
```

### 关键确认点

- 页面类型：首页、定价页、功能页、登录页、管理后台、数据看板、CRM、设置页。
- 内容结构：Hero、客户 Logo、功能卡片、步骤说明、案例、FAQ、CTA。
- PC 宽度：1440、1280、1024 或自定义。
- 响应式：是否需要移动端、平板端。
- 技术偏好：纯 HTML、React、Next.js 风格组件、Tailwind、CSS Modules。
- 交互：导航、下拉菜单、筛选、表格分页、弹窗、表单验证、图表 tooltip。

### 交付物

```text
web-project/
  site_map.json
  design_tokens.json
  pages/home/reference_desktop.png
  pages/home/layout.json
  src/components/
  src/pages/
  src/styles/
  assets/
  qa/visual-regression/
```

## 3. 系统架构

### 3.1 总体架构

```text
Frontend Web App
  ├─ Onboarding / Project Wizard
  ├─ Multi-turn Chat Workspace
  ├─ Outline & Style Review
  ├─ Image Gallery & Annotation Canvas
  ├─ Editable Preview
  └─ Export Center
        │
        ▼
Convex Backend
  ├─ Auth & Workspaces
  ├─ Projects / Conversations / Messages
  ├─ Generation Jobs & Status
  ├─ Layout JSON Store
  ├─ Asset Metadata Store
  ├─ Human Review Queue
  └─ Export Artifacts
        │
        ▼
AI Orchestration Layer
  ├─ ChatGPT Text Model: 需求澄清、大纲、结构化 JSON、修复建议
  ├─ ChatGPT Vision/Text Model: 图片理解、对象分层、QA 对比描述
  ├─ Image Model: 效果图生成、局部重绘、风格变体
  └─ Tool Workers: PPTX/HTML/React/Prefab 构建与截图验证
        │
        ▼
Artifact Storage
  ├─ reference images
  ├─ cropped assets
  ├─ layout.json
  ├─ previews
  ├─ validation reports
  └─ final exports
```

### 3.2 OpenAI 能力分工

- **文本模型**：负责多轮需求澄清、产品简报、PPT 大纲、页面信息架构、风格规范、结构化 `layout.json` 草案、交互说明和修复计划。
- **视觉理解模型**：负责读取效果图，识别文本、组件、层级、bbox、颜色、z-index、交互热点和复原策略。
- **图像模型**：负责从大纲和风格生成效果图，也负责局部图像编辑、风格变体和复杂资产补全。
- **工具调用/后台任务**：负责把 JSON 转成实际 PPTX、HTML、React 或 Prefab，并执行截图、OCR、视觉差异和交互测试。

官方 OpenAI 文档中的 Responses API 支持文本和图像输入，并可用于状态化、多步骤、带工具调用的模型交互；图像生成也可以作为对话或多步骤流程的一部分。因此本产品应优先把“多轮对话、图像生成、结构化输出、工具构建”设计为统一编排流程，而不是多个孤立接口。

### 3.3 Convex 后端职责

Convex 适合作为实时产品状态与任务编排层：

- 管理用户、团队、项目、权限。
- 保存对话、确认项、结构化 brief、outline、style guide。
- 保存生成任务状态：queued、running、needs_review、failed、completed。
- 保存页面/屏幕/组件级 `layout.json`。
- 管理资产元数据和最终导出文件记录。
- 向前端实时推送生成进度、局部预览和错误信息。
- 调度 actions 调用 OpenAI、渲染服务、导出服务和 QA 服务。

## 4. Convex 数据模型草案

```ts
users: {
  name: string;
  email: string;
  image?: string;
}

workspaces: {
  name: string;
  ownerId: Id<'users'>;
  plan: 'free' | 'pro' | 'team';
}

projects: {
  workspaceId: Id<'workspaces'>;
  type: 'ppt' | 'app' | 'web_pc';
  title: string;
  status: 'briefing' | 'outlining' | 'styling' | 'imaging' | 'rebuilding' | 'exported';
  targetExport: 'pptx' | 'html' | 'react' | 'prefab';
  createdAt: number;
  updatedAt: number;
}

messages: {
  projectId: Id<'projects'>;
  role: 'user' | 'assistant' | 'system' | 'tool';
  content: string;
  structured?: unknown;
  createdAt: number;
}

projectBriefs: {
  projectId: Id<'projects'>;
  audience?: string;
  goal: string;
  constraints: string[];
  accepted: boolean;
}

outlines: {
  projectId: Id<'projects'>;
  version: number;
  pages: Array<{ id: string; title: string; purpose: string; keyPoints: string[] }>;
  accepted: boolean;
}

styleGuides: {
  projectId: Id<'projects'>;
  version: number;
  mood: string;
  colors: Record<string, string>;
  fonts: Array<{ role: string; family: string }>;
  density: 'low' | 'medium' | 'high';
  accepted: boolean;
}

screens: {
  projectId: Id<'projects'>;
  index: number;
  name: string;
  sourceImageAssetId?: Id<'assets'>;
  previewImageAssetId?: Id<'assets'>;
  layoutId?: Id<'layouts'>;
  status: 'planned' | 'image_ready' | 'layout_ready' | 'validated' | 'needs_review';
}

layouts: {
  projectId: Id<'projects'>;
  screenId: Id<'screens'>;
  schemaVersion: string;
  json: unknown;
  validationSummary?: unknown;
  createdAt: number;
}

assets: {
  projectId: Id<'projects'>;
  kind: 'reference' | 'crop' | 'preview' | 'export' | 'qa';
  storageId: string;
  mimeType: string;
  width?: number;
  height?: number;
  metadata?: unknown;
}

jobs: {
  projectId: Id<'projects'>;
  type: 'brief' | 'outline' | 'style' | 'image' | 'layout' | 'export' | 'qa' | 'repair';
  status: 'queued' | 'running' | 'failed' | 'completed' | 'needs_review';
  input: unknown;
  output?: unknown;
  error?: string;
  createdAt: number;
  updatedAt: number;
}
```

## 5. 核心 AI 工作流

### 5.1 点子到 Brief

输入：用户一句话、材料上传、参考链接或已有图片。

输出：结构化 brief。

```json
{
  "project_type": "ppt",
  "goal": "为投资人展示 AI 设计工具商业计划",
  "audience": "早期投资人",
  "tone": "专业、可信、科技感",
  "required_outputs": ["editable_pptx", "preview_images"],
  "open_questions": ["期望页数是多少？", "是否已有品牌色？"]
}
```

前端体验：AI 给出 3-5 个关键问题，用户可以逐条确认，也可以点击“使用推荐答案”。

### 5.2 Brief 到大纲/页面清单

根据项目类型生成不同结构：

- PPT：章节和逐页标题。
- App：核心用户流和 screen list。
- Web/PC：站点地图、页面区块、响应式断点。

每一页/屏都应包含：目的、主要内容、视觉建议、需要的素材、交互需求。

### 5.3 大纲到风格指南

生成可编辑风格规范：

```json
{
  "mood": "modern SaaS, clean, high trust",
  "colors": {
    "primary": "#2563EB",
    "background": "#F8FAFC",
    "text": "#0F172A"
  },
  "fonts": [
    { "role": "heading", "family": "Inter" },
    { "role": "body", "family": "Inter" }
  ],
  "radii": { "card": 16, "button": 12 },
  "density": "medium"
}
```

### 5.4 风格指南到效果图

每页/屏单独生成参考图，避免多页拼图导致细节丢失：

```text
page brief + style guide + target size
  -> image prompt
  -> reference.png
  -> thumbnail/contact sheet
  -> user approval or revision
```

用户在这一步只判断视觉方向：是否喜欢、是否符合品牌、信息是否完整、是否需要换风格。

### 5.5 效果图到 Layout JSON

AI 对确认后的效果图进行对象分层：

- 文本：内容、bbox、字号、颜色、字重、对齐。
- 形状：卡片、按钮、线条、表格、图表容器。
- 资产：照片、插画、图标、纹理、复杂图表切片。
- 交互：点击区域、跳转、状态切换、弹窗。
- QA：置信度、已知限制、需要人工确认的对象。

### 5.6 Layout JSON 到目标产物

```text
layout.json
  ├─ PPTX Builder: PowerPoint 原生对象 + 局部图片资产
  ├─ HTML Builder: DOM + CSS + TypeScript interactions
  ├─ React Builder: TSX components + tokens + assets + router hooks
  └─ Prefab Builder: 节点树 + Sprite/Label/Button/Layout + binding script
```

### 5.7 验证与修复

每次导出后生成：

- `preview.png`：目标产物重新渲染截图。
- `validation.json`：文本完整性、资产完整性、视觉差异、交互测试结果。
- `repair_queue.json`：只包含失败对象，支持局部重试。

## 6. 前端产品信息架构

### 6.1 主导航

- Dashboard：项目列表、最近导出、模板入口。
- New Project：创建 PPT、App、Web/PC 三类项目。
- Chat Workspace：多轮对话和结构化确认。
- Visual Board：效果图、版本、批注。
- Rebuild Studio：layout、组件树、资产、交互热点。
- Export Center：PPTX、HTML、React、Prefab、ZIP。

### 6.2 创建项目向导

三张入口卡片：

1. **创建 PPT**
   - 输入主题、材料、页数、受众。
   - 输出可编辑 PPTX。
2. **创建 App 设计**
   - 输入 App 点子、目标用户、核心流程。
   - 输出可点击移动端 Demo 或 React 组件。
3. **创建 Web/PC 设计**
   - 输入业务类型、页面类型、内容诉求。
   - 输出 Web 页面、React 组件或可开发项目。

### 6.3 多轮对话界面

左侧：聊天和 AI 建议。  
右侧：结构化确认面板。

确认面板包含：

- Brief。
- Outline / Screen list / Site map。
- Style guide。
- Interaction map。
- Export settings。

用户可以直接编辑结构化字段，而不是只能在聊天里描述。

### 6.4 效果图确认界面

- 画廊：查看所有页面/屏幕。
- 单页对比：当前版本、上一版本、参考风格。
- 批注：圈选局部并输入修改意见。
- 版本：保留每次生成的 prompt、图片、用户反馈。
- 通过按钮：只有通过的效果图进入可编辑重建。

### 6.5 可编辑/可交互预览界面

- PPT：内嵌幻灯片预览、对象清单、可编辑率统计。
- App：手机壳预览、点击路径、状态切换。
- Web/PC：桌面画布、响应式宽度切换、组件树。
- Prefab：节点树、资源清单、绑定脚本和引擎导出包。

## 7. MVP 范围

### 7.1 MVP 必做

- 三类项目入口：PPT、App、Web/PC。
- Convex 项目、对话、任务、资产、layout 存储。
- ChatGPT 文本模型生成 brief、大纲、风格指南。
- 图像模型生成单页/单屏效果图。
- 用户可批注效果图并触发局部修改。
- 效果图转 `layout.json`。
- PPTX 导出和 HTML Demo 导出。
- 基础 QA：预览截图、文本完整性、禁止整页贴图检查。

### 7.2 MVP 暂缓

- 完整 React 项目工程化导出。
- Unity/Godot/Cocos 多引擎 Prefab 同时支持。
- 自动复杂图表数据反推。
- 完整设计系统同步 Figma。
- 多人实时协作和评论权限。
- 付费、团队空间、模板市场。

## 8. 分阶段路线图

### Phase 0：基础架构

- 建立 Convex schema、auth、storage、jobs。
- 建立 OpenAI 调用封装和任务状态机。
- 建立 artifact 存储目录规范。
- 建立 `layout.json` schema v1。

### Phase 1：PPT 从点子到可编辑

- 点子到 PPT brief。
- PPT 大纲和逐页设计 prompt。
- 逐页效果图生成。
- 效果图转 PPT layout。
- PPTX builder。
- 预览和局部修复。

### Phase 2：App 从点子到可点击 Demo

- App brief、screen list、user flow。
- 移动端效果图生成。
- 点击热点标注。
- HTML/CSS/TS demo builder。
- Playwright 点击测试。

### Phase 3：Web/PC 从点子到 React

- Web site map、页面区块、响应式断点。
- PC 效果图生成。
- HTML builder 升级为 React builder。
- 组件抽取、tokens 输出、路由和表单交互。

### Phase 4：Prefab 与高级可编辑能力

- Cocos 或 Unity 单引擎优先。
- Sprite 切片、九宫格、图集标签。
- 节点树和绑定脚本生成。
- 引擎内预览和截图 QA。

## 9. 关键质量指标

| 指标 | 定义 | MVP 目标 |
| --- | --- | --- |
| Brief 确认轮数 | 从点子到结构化 brief 的平均轮数 | 3 轮以内 |
| 效果图通过率 | 用户无需重生即可接受的页面比例 | 60%+ |
| 可编辑率 | 文本、按钮、卡片、表格等可编辑对象占比 | 70%+ |
| 整页贴图率 | 最终产物中整页/整屏图片背景占比 | 5% 以下 |
| 导出成功率 | job completed / job started | 90%+ |
| 视觉回归差异 | preview 与 reference 的主要区域差异 | 按页面类型分级 |
| 交互通过率 | 点击、跳转、弹窗、Tab 测试通过率 | 80%+ |

## 10. 风险与应对

| 风险 | 表现 | 应对 |
| --- | --- | --- |
| 用户需求模糊 | 直接生成效果图质量不稳定 | 强制 brief 和大纲确认 |
| 图像文字不准 | 效果图中文字错字或不可读 | 文字由文本模型确定，图像只负责视觉参考；重建时以 confirmed text 为准 |
| 过度依赖截图 | 导出不可编辑 | QA 检查整页贴图和可编辑率 |
| 多平台构建复杂 | PPT/HTML/React/Prefab 差异大 | 统一 layout schema，构建器分层实现 |
| 视觉差异难量化 | 用户主观觉得不像 | 引入 preview 对比、局部批注和 repair queue |
| 成本不可控 | 多轮图像生成消耗高 | 先低清草图，再高清通过图；缓存 prompt 和图片版本 |
| 交互推断错误 | AI 猜错按钮行为 | 多屏关系 + 用户标注 + 结构化 interaction map |

## 11. 推荐的下一步执行清单

1. 定义 `layout.json` v1 最小 schema：canvas、tokens、assets、elements/components、interactions、qa。
2. 在 Convex 中实现 projects、messages、briefs、outlines、styleGuides、screens、layouts、assets、jobs 表。
3. 实现三类项目创建向导和多轮确认 UI。
4. 实现 text model orchestration：idea -> brief -> outline -> style guide。
5. 实现 image generation orchestration：page prompt -> reference image -> revision。
6. 实现 screenshot-to-layout worker：reference image -> layout.json。
7. 先实现 PPTX builder 与 HTML builder。
8. 实现预览截图和 validation report。
9. 以 3 个样例模板测试：商业 PPT、移动 App、电商 Web 首页。
10. 再扩展 React builder 与 Prefab builder。
