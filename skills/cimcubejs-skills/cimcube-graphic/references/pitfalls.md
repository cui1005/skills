# 真实代码提醒 + 误区对照

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。

## 一、真实代码提醒（运行时核验）

1. `Glodon.CIMCube.Graphic` 导出对象**没有 BaseGraphic/ModelGraphic 两个键**；写 `new G.ModelGraphic(...)` 会报错。文档中带模型/GLTF 的需求请走 `cimcube-layer` 的模型图层（如 ModelLayer / GraphicLayer 绘制 `type='model'` 的内部实现）。
2. 底层全部渲染为 **Entity + CustomDataSource**，因此拾取命中对象是 `Cesium.Entity`（`e.picked.id`），Entity 的 `id` 即图元 id；批量几千级图元请评估 Entity 渲染开销。
3. 图元业务数据尽量用 `addProperty/getProperty/getAllProperties`，不塞 Entity 扩展字段。
4. `setStyle` 里点/线/标签的 LOD 字段既可直接给数组（`distanceDisplayCondition:[near,far]`、`scaleByDistance:[near,nearValue,far,farValue]`），也支持 `xxx_near/_nearValue/_far/_farValue` 扁平写法（详细键看各 API_DOC 页）。已废弃的 `setOptions` **不要再写**。
5. 不要用 `graphic.update()` / `graphic.setPosition()` 等**不存在**的刷新方法；改完样式调用对应 `set*` 即可，渲染会自动更新。

## 二、常见问题 / 误区对照

| 易错写法 / 常见说法 | 真相（API_DOC + 运行时） |
|---|---|
| `position:{x:lon,y:lat}`、`style:{...}` 大对象 | `position/positions` 是**数值数组**；样式优先写顶层键，`style` 只在点/标/面/线类可选 |
| `HorizontalOrigin: 0=左`、`VerticalOrigin:0=下`、`CornerType 0=尖` | 数值与语义见 SKILL.md §三，方向/角型勿记反（`VerticalOrigin.TOP = -1`） |
| Polygon 贴地默认 false / Polyline 贴地默认 true | 正好相反：Polygon `clampToGround` 默认 **true**；Polyline 构造**无**该键 |
| Corridor `width` 总宽度像素 / `radius` | `width` = **缓冲半径（米）**，必填 |
| `layer.add/remove/removeById/update/getGraphicsByCondition` | 无此 API；真名 `addGraphic/removeGraphic/removeAllGraphics/getGraphicById/getAllGraphics` |
| `ModelGraphic`、`BaseGraphic` 可直接 new | 未导出（无命名空间键）；ModelGraphic 有内部实现但不对外 |
| `new PolygonGraphic({west,south,east,north})` / `new EllipseGraphic({center,radius})` / `BoxGraphic({center,...})` | Rectangle 用**对角 positions**、Ellipse 用 `semiMajorAxis/semiMinorAxis/height`、Box 用 `position` + `dimensions{length,width,height}` |
| 构造即有 `cesiumEntity` | 需要 `layer.addGraphic`/`addToViewer(dataSource)` 后才真正出现在场景 |
| `graphic.setOptions({...})` 改样式 | 部分图形下已废弃，**勿写**。改样式用 `setStyle(xxxStyleOptions)`；`Corridor/Rectangle/Ellipse/Cylinder/Ellipsoid/Box/Plane/PolylineVolume` 无该接口，暂时使用setOptions或走构造 opts |
| `setStyle` 传普通 Object | 传对应 `PointStyleOptions/LabelStyleOptions/...`，与 user-guide 405 的 FeatureStyle 同构 |
| 给 `CorridorGraphic`/`CylinderGraphic`/`EllipseGraphic`/`EllipsoidGraphic`/`RectangleGraphic` 调 `setStyle` | 运行时无此方法（会 TypeError）；这几个类的样式只能在构造 opts 给全 |
