# 公共契约 — 导出真相 · 图元公共能力 · 与 GraphicLayer 的边界

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。

## 一、导出真相（运行时 vs API_DOC）

| 类别 | 内容 |
|---|---|
| **运行时导出 21 键** | 14 具体图元类：`PointGraphic`、`BillboardGraphic`、`LabelGraphic`、`PolygonGraphic`、`PolylineGraphic`、`PolylineVolumeGraphic`、`CorridorGraphic`、`RectangleGraphic`、`EllipseGraphic`、`WallGraphic`、`BoxGraphic`、`CylinderGraphic`、`EllipsoidGraphic`、`PlaneGraphic`；5 枚举：`CornerType`、`HeightReference`、`HorizontalOrigin`、`VerticalOrigin`、`LabelStyle`；2 坐标类：`GeoPoint`、`TwoDimensionPoint` |
| **API_DOC 有页但运行时未导出** | `BaseGraphic`（所有图元的抽象基类：类型标注/内部继承用）、`ModelGraphic`（内部实现存在，`drawType='model'`，供内部绘制/编辑用；**不能 `new G.ModelGraphic()`**）。需要往场景挂 GLTF 模型时走图层体系（cimcube-layer） |
| 几何本质 | 每个图元对象包装 **1 个 Cesium Entity**；`GraphicLayer` 内部维护 `CustomDataSource`，图元经 `addToViewer(dataSource)`/`removeSelf(dataSource)` 挂/摘 |

## 二、图元公共能力（全部图元继承自 BaseGraphic）

- `addProperty(key, value)` / `getProperty(key)` / `removeProperty(key)` / `getAllProperties()`（返回 Map）：给图元挂业务属性，pick 时随图形一起拿到。
- `getEntity()`：底层 Cesium Entity；`flyTo()`：相机定位到图形。
- `show = false/true` 显隐；`id`/`name`（只读）；`dynamic`（是否动态绘制——引擎做标绘预览时内部置 true）。
- `addToViewer(dataSource)` / `removeSelf(dataSource)`：手动挂载/摘除（一般经 `layer.addGraphic` 间接调用）。
- 事件本体：GraphicLayer 的拾取/进出事件携带 `picked.id`（Entity 的 id = 图元 id），经 `LayerEventType` 触发，事件归属详表见 `cimcube-layer`。

## 三、坐标数据类

```js
// GeoPoint：地理坐标（经纬度+椭球高），new GeoPoint(lon, lat, height)；成员 lon/lat/height
const gp = new G.GeoPoint(116.39, 39.905, 20);   // gp.lon / gp.lat / gp.height
// TwoDimensionPoint：平面/像素坐标，new TwoDimensionPoint(x, y)；成员 x/y
const tdp = new G.TwoDimensionPoint(10, -5);     // 例：pixelOffset:[10,-5] 等价
// PolylineVolumeGraphic 的 shape 需要 Array<TwoDimensionPoint>
const shape = [new G.TwoDimensionPoint(-1, 1), new G.TwoDimensionPoint(1, 1),
               new G.TwoDimensionPoint(1, -1), new G.TwoDimensionPoint(-1, -1)];
```

## 四、与 GraphicLayer 的边界（增删查 / 事件 / 绘制编辑归属 cimcube-layer）

GraphicLayer 完整的**增删查、图层参数、pick/事件、以及 draw 交互绘制**属于 `cimcube-layer` 技能（GraphicLayer 类页在其域下）。与图元直接相关的部分（均经运行时核实）：

- 增删查公开方法：`addGraphic(graphic)` / `removeGraphic(graphic)` / `removeAllGraphics()` / `getGraphicById(id)` / `getAllGraphics()` / `flyTo()`。**不存在** `add/update/removeById/getGraphicsByCondition` 等名称。
- 加入成功触发 `GRAPHIC_ADDED`（事件在 layer，参数为图形）；`removeGraphic` 内部调用 `graphic.removeSelf(dataSource)`。
- 拾取/悬停：`ENTITY_CLICK`、`ENTITY_MOUSE_ENTER`、`ENTITY_MOUSE_LEAVE`（`LayerEventType`，鼠标移动类事件由 BaseLayer `enableMouseMove` 控制）；用 `layer.on(...)` 注册。
- 标绘/编辑：`startDraw({ type:Glodon.CIMCube.Global.Defines.GraphicType.X, style, config, normalize: false })`、`stopDraw()`、`clearDraw()`、`startEditing(graphic)`、`stopEditing()`、`setTipsMessage()`、`setCursor()`、`allowEmptySelection`（编辑态点空白是否取消，默认 true）。`normalize:true` 时 `style` 走 user-guide 405 的 FeatureStyle；`GraphicType` 常量值与 DrawControl 注册的绘制类型一一对应（矩形/墙等图元的只读 `drawType` 亦据此生成）。创建 GraphicLayer、`startDraw` 支持的类型全集与绘制事件 → 看 `cimcube-layer`。
