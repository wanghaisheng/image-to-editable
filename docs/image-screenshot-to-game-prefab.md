# 图片效果图到游戏引擎 Prefab 的复原文档

本文定义从 UI 效果图图片复原为游戏引擎 Prefab 的流程。它适用于 Cocos Creator、Unity UGUI、Godot Control 等 UI 系统。推荐使用中间 `layout.json` 描述节点树，再生成 Prefab，并通过 TypeScript/C# 绑定脚本连接交互和数据。

## 1. 目标产物

```text
prefab-demo/
  layout/
    shop_panel.layout.json
  assets/
    ui/shop_panel/bg_panel.png
    ui/shop_panel/icon_coin.png
    ui/shop_panel/btn_buy_normal.png
  prefabs/
    ShopPanel.prefab
  scripts/
    ShopPanelBinding.ts
  qa/
    prefab_preview.png
    validation.json
```

目标：

- 视觉接近效果图。
- UI 节点可编辑：Label、Button、Sprite、Layout、ScrollView、Mask 等尽量使用引擎原生组件。
- 图片资源独立切分，便于图集打包。
- 按钮、列表项、弹窗、状态切换通过绑定脚本可交互。
- Layout JSON 可重复生成 Prefab，便于修复和版本升级。

## 2. 引擎无关 UI 抽象

| Layout JSON 类型 | Cocos Creator | Unity UGUI | Godot |
| --- | --- | --- | --- |
| `node` | `Node` | `GameObject` | `Control` |
| `text` | `Label` | `TextMeshProUGUI` | `Label` |
| `image` | `Sprite` | `Image` | `TextureRect` |
| `button` | `Button + Sprite + Label` | `Button + Image + TMP` | `Button` |
| `panel` | `Sprite/Widget/Layout` | `Image/LayoutGroup` | `PanelContainer` |
| `scroll` | `ScrollView` | `ScrollRect` | `ScrollContainer` |
| `mask` | `Mask` | `Mask/RectMask2D` | `Clip Contents` |
| `layout` | `Layout` | `LayoutGroup` | `Container` |

## 3. Prefab Layout JSON

```json
{
  "schema_version": "1.0",
  "engine_target": "cocos_creator_3.x",
  "source": {
    "path": "input/shop_panel.png",
    "width": 1334,
    "height": 750
  },
  "canvas": {
    "design_resolution": [1334, 750],
    "anchor": [0.5, 0.5],
    "scale_mode": "show_all"
  },
  "assets": [
    {
      "id": "icon_coin",
      "type": "sprite",
      "path": "assets/ui/shop_panel/icon_coin.png",
      "source_bbox": [1040, 34, 44, 44],
      "packing_tag": "ui_shop"
    }
  ],
  "nodes": [
    {
      "id": "root",
      "name": "ShopPanel",
      "type": "panel",
      "parent": null,
      "bbox": [0, 0, 1334, 750],
      "anchor": [0.5, 0.5],
      "components": ["Widget"]
    },
    {
      "id": "btn_buy",
      "name": "BtnBuy",
      "type": "button",
      "parent": "root",
      "bbox": [1020, 620, 220, 72],
      "anchor": [0.5, 0.5],
      "style": { "normal_sprite": "btn_buy_normal", "radius": 18 },
      "children": ["label_buy"],
      "binding": { "event": "click", "handler": "onBuyClicked" }
    },
    {
      "id": "label_buy",
      "name": "LabelBuy",
      "type": "text",
      "parent": "btn_buy",
      "text": "购买",
      "bbox": [1075, 638, 110, 36],
      "style": { "font_size": 30, "color": "#FFFFFF", "bold": true }
    }
  ],
  "bindings": [
    { "node_id": "btn_buy", "property": "click", "handler": "onBuyClicked" },
    { "node_id": "coin_label", "property": "text", "data_path": "player.coin" }
  ]
}
```

## 4. 坐标转换

效果图一般以左上角为原点，引擎 UI 常以中心或节点锚点为原点。需要统一转换：

```text
source bbox: [x, y, w, h]
canvas: [W, H]

centerX = x + w / 2
centerY = y + h / 2
engineX = centerX - W / 2
engineY = H / 2 - centerY
size = [w, h]
```

节点锚点建议：

- 全屏根节点：`anchor = [0.5, 0.5]`。
- 背景铺满：加 Widget/Stretch。
- 列表项：`anchor = [0.5, 0.5]`，由 Layout 组件管理。
- 角标/货币栏：用 Widget pin 到上/右/下。

## 5. 复原流程

### Step 1：UI 语义识别

识别：

- 根 Canvas / Panel。
- 背景图、弹窗面板、遮罩层。
- Button、Toggle、Slider、Input、ScrollView、Tab。
- Label、RichText、BitmapFont 数字。
- 图标、道具格、头像、品质框。
- 列表和网格：背包、商城、任务、排行榜。

### Step 2：资产拆分

游戏 UI 对资源复用要求高，资产拆分比 HTML 更严格：

- 可九宫格的面板/按钮导出为 `sliced sprite`，记录 `border`。
- 重复图标只保留一份，多个节点引用同一 asset id。
- 大背景保留为独立纹理，建议压缩格式和图集标签。
- 小图标进入 atlas，记录 packing tag。
- 动态数字可映射 BitmapFont 或 Label。

### Step 3：节点树归纳

不要把所有对象平铺到 root。应归纳父子关系：

```text
ShopPanel
  ├─ BgDim
  ├─ Window
  │   ├─ TitleBar
  │   ├─ CurrencyBar
  │   ├─ GoodsScrollView
  │   │   └─ Content
  │   │       ├─ GoodsItem_001
  │   │       └─ GoodsItem_002
  │   └─ BtnClose
  └─ ToastLayer
```

归纳规则：

- bbox 包含关系强的对象成为父子。
- 文本属于最近的按钮/卡片/列表项。
- 滚动内容区域下的重复项归为 prefab item。
- 弹窗浮层独立为 overlay 节点。

### Step 4：生成 Prefab

构建器职责：

1. 读取 `layout.json`。
2. 导入/登记 assets。
3. 创建节点树。
4. 添加 Sprite、Label、Button、Layout、ScrollView、Mask 等组件。
5. 设置坐标、尺寸、锚点、层级。
6. 添加绑定脚本并把节点引用写入脚本属性。
7. 保存 Prefab。

## 6. Prefab 绑定 TypeScript 示例

以 Cocos Creator 为例：

```ts
import { _decorator, Component, Label, Button, Node } from 'cc';
const { ccclass, property } = _decorator;

@ccclass('ShopPanelBinding')
export class ShopPanelBinding extends Component {
  @property(Button)
  btnBuy: Button | null = null;

  @property(Label)
  coinLabel: Label | null = null;

  @property(Node)
  goodsContent: Node | null = null;

  private coin = 0;

  onLoad() {
    this.btnBuy?.node.on(Button.EventType.CLICK, this.onBuyClicked, this);
  }

  bindData(data: { player: { coin: number } }) {
    this.coin = data.player.coin;
    if (this.coinLabel) this.coinLabel.string = String(this.coin);
  }

  onBuyClicked() {
    this.node.emit('shop-buy-clicked');
  }
}
```

对应 JSON 绑定：

```json
{
  "script": "ShopPanelBinding",
  "properties": {
    "btnBuy": "btn_buy",
    "coinLabel": "coin_label",
    "goodsContent": "goods_content"
  },
  "events": [
    { "node": "btn_buy", "component": "Button", "event": "CLICK", "handler": "onBuyClicked" }
  ]
}
```

## 7. QA 与验收

| 检查项 | 验收标准 |
| --- | --- |
| 视觉 | Prefab 运行截图与源图主要布局一致 |
| 节点 | 关键 UI 不是单张整图，而是可编辑节点 |
| 资产 | 资源有唯一 id、路径、hash、图集标签 |
| 坐标 | 锚点、父子关系、适配策略正确 |
| 交互 | Button/Toggle/ScrollView 在运行时可用 |
| 绑定 | 脚本属性引用不为空，事件可触发 |
| 性能 | 小图标入 atlas，大图不过度切碎 |

## 8. MVP 与演进

MVP：

- 单张 UI 图 → 单个 Prefab。
- Label、Sprite、Button 三类节点。
- 绝对坐标和中心锚点。
- 手动标注按钮事件。
- 生成一个绑定脚本模板。

演进：

- 自动识别 ScrollView、Grid、Tab、弹窗。
- 自动生成 item prefab 和数据绑定。
- 自动九宫格切片与图集配置。
- 多分辨率适配规则。
- 引擎内截图回归测试。
