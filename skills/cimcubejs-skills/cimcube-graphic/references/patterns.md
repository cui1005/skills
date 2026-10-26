# 快速上手 + 常见模式

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。

## 快速上手（添加 → 事件 → 删除）

```js
const G = Glodon.CIMCube.Graphic;
const map = new Glodon.CIMCube({ id: "container", gisResourceUrl: "./dist/" });
// 1) 拿到 GraphicLayer（创建/获取图层见 cimcube-layer，例如 map.layerManager.getLayer("graphicLayer")）
const layer = map.layerManager.addLayer({
  type: Glodon.CIMCube.Global.Defines.LayerType.GRAPHIC, id: "graphicLayer", name: "graphic",
});
// 2) 构造图元：顶层选项，position/positions 为数值数组
const pt = new G.PointGraphic({
  id: "pt01", name: "监测点",
  position: [116.391, 39.905, 12],   // [经度, 纬度, 高度(米)]
  pixelSize: 12, color: "#ffd400", outlineColor: "#000000", outlineWidth: 2,
  heightReference: G.HeightReference.CLAMP_TO_GROUND,
  distanceDisplayCondition: [0, 5000],
});
const poly = new G.PolygonGraphic({
  id: "pg01", name: "用地",
  positions: [[116.39,39.90,0],[116.41,39.90,0],[116.41,39.92,0],[116.39,39.92,0]],
  material: "#3a9eff", materialType: "color", outline: true, outlineColor: "#0a5ac2", outlineWidth: 1,
  clampToGround: true,
});
// 3) 加入图层
layer.addGraphic(pt); layer.addGraphic(poly);
// 4) 事件（在 layer 上，LayerEventType）
layer.on(Glodon.CIMCube.Global.Defines.LayerEventType.GRAPHIC_ADDED, (e) => console.log("added", e.arguments));
layer.on(Glodon.CIMCube.Global.Defines.LayerEventType.ENTITY_CLICK, (e) => {
  const g = e.picked && e.picked.id;          // 拾取命中 → 图形（含 id）
  if (g && g.id === "pt01") { /* 自定义属性、高亮、编辑等 */ }
});
layer.on(Glodon.CIMCube.Global.Defines.LayerEventType.ENTITY_MOUSE_ENTER, (e) => { /* e.picked.id */ });
// 5) 查/删
const found = layer.getGraphicById("pt01");    // BaseGraphic|null
const all = layer.getAllGraphics();            // BaseGraphic[]
layer.removeGraphic(found);
layer.removeAllGraphics();
```

## 常见模式

> 以下片段默认已存在 `GraphicLayer` 实例 `layer`（由上方快速上手创建）。涉及图元 `pt` 的片段已自带创建语句，可独立运行。

**① 挂业务属性并在拾取时读取**

```js
// pt 与「快速上手」中为同一监测点；独立运行时先创建：
const pt = new Glodon.CIMCube.Graphic.PointGraphic({ id:"pt01", position:[116.391,39.905,12], pixelSize:12, color:"#ffd400", outlineColor:"#000000", outlineWidth:2 });
pt.addProperty("deviceId", "D-1001");
pt.addProperty("status", "online");
layer.on(Glodon.CIMCube.Global.Defines.LayerEventType.ENTITY_CLICK, (e) => {
  const g = e.picked && e.picked.id;
  if (g) console.log(g.getProperty("deviceId"), g.getAllProperties());
});
```

**② 悬停高亮 / 移出还原（面、线）**

```js
layer.on(Glodon.CIMCube.Global.Defines.LayerEventType.ENTITY_MOUSE_ENTER, (e) => {
  const g = e.picked && e.picked.id;
  if (g instanceof Glodon.CIMCube.Graphic.PolygonGraphic) g.setHighlight({ fillColor: "rgba(255,212,0,0.85)" });
});
layer.on(Glodon.CIMCube.Global.Defines.LayerEventType.ENTITY_MOUSE_LEAVE, () => layer.getAllGraphics().forEach(g => {
  if (g instanceof Glodon.CIMCube.Graphic.PolygonGraphic) g.setHighlight({ fillColor: "rgba(58,158,255,0.5)" });
}));
```

**③ 点 + 标签联动**

```js
const pt = new G.PointGraphic({ id: "pt1", position: [116.39, 39.90, 0], pixelSize: 10, color: "#ffd400" });
const lb = pt.setLabel({ text: "泵站 #1", font: "14px sans-serif", fillColor: "#ffffff" });
layer.addGraphic(pt); layer.addGraphic(lb);
```

**④ 相机定位到图元**

```js
const pt = new Glodon.CIMCube.Graphic.PointGraphic({ id:"pt01", position:[116.391,39.905,12], pixelSize:12, color:"#ffd400" });
layer.addGraphic(pt);            // 先确保该图元已加入图层
const g = layer.getGraphicById("pt01");
g.flyTo();                       // 定位到该图形
```

**⑤ 显隐与批量清理**

```js
const pt = new Glodon.CIMCube.Graphic.PointGraphic({ id:"pt01", position:[116.391,39.905,12], pixelSize:12, color:"#ffd400" });
layer.addGraphic(pt);
pt.show = false;                             // 隐藏
layer.getAllGraphics().forEach(g => g.show = true);
layer.removeAllGraphics();
```
