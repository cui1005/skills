# 图层事件系统

入口 `map.layerManager`（事件挂在图层实例上）。

## 订阅方式（⚠️ 必须用数值枚举）

```js
const D = Glodon.CIMCube.Global.Defines;
layer.on(D.LayerEventType.LOADED, handler);                 // ✅ 正确
layer.on(D.LayerEventType.ENTITY_CLICK, (e) => {            // ✅
  console.log(e.arguments.pickId, e.arguments.featureInfo);
});
// layer.on("LOADED", handler);                             // ❌ 字符串不触发（事件 type 是数字）
```

> 鼠标/屏幕事件亦可用 `Defines.ScreenSpaceEventType.LEFT_CLICK` 等；两枚举在鼠标事件上取值一致。
> `on(type, listener, thisArg)` 返回取消函数；`off(type, listener?)` 省略 listener 时移除该类型全部监听（见 `references/base-layer.md`）。

## `Defines.LayerEventType` 取值

| 事件 | 值 | 触发场景 |
|---|---|---|
| `LEFT_DOWN` `LEFT_UP` `LEFT_CLICK` `LEFT_DOUBLE_CLICK` | 0 1 2 3 | 鼠标 |
| `RIGHT_DOWN` `RIGHT_UP` `RIGHT_CLICK` | 5 6 7 | 鼠标 |
| `MIDDLE_DOWN` `MIDDLE_UP` `MIDDLE_CLICK` | 10 11 12 | 鼠标 |
| `MOUSE_MOVE` `WHEEL` `MOUSE_LEAVE` | 15 16 20 | 鼠标（需 `enableMouseMove`） |
| `PINCH_START` `PINCH_END` `PINCH_MOVE` | 17 18 19 | 触控 |
| `MOUSE_SNAP` | 23 | 吸附 |
| `ENTITY_CLICK` | 1001 | 点选实体（`e.arguments` 含 `pickId`/`featureInfo`/`pickEntity`/`geometry`） |
| `ENTITY_MOUSE_ENTER` `ENTITY_MOUSE_LEAVE` | 1002 1003 | 实体悬停进出 |
| `GRAPHIC_DELETED` `GRAPHIC_ADDED` | 1004 1005 | 图元增删 |
| `VISIBLE` | 1006 | 瓦片可见性变化 |
| `LOADED` | 1007 | 图层数据加载完成 |
| `TRANSFORM_CHANGED` | 1008 | 图层变换改变 |
| `TILELOADED` | 1009 | 瓦片加载完成 |
| `PRECLUSTER` `POSTCLUSTER` | 1010 1011 | 聚合前/后 |
| `HIGHLIGHT` `DESELECTION` | 1012 1013 | 高亮 / 取消选择 |

## 各图层可用事件

| 事件 | 图层 |
|---|---|
| `LOADED` | Tiles3D / Gltf / GSMM / I3S / S3M / USD / GeoJson / Gml |
| `TILELOADED` | Tiles3D |
| `VISIBLE` | Tiles3D |
| `TRANSFORM_CHANGED` | Tiles3D / Gltf / GSMM / I3S / S3M / TransformableGroup |
| `ENTITY_CLICK` | GeoJson / Gml / Kml / Graphic / Cluster |
| `ENTITY_MOUSE_ENTER` / `ENTITY_MOUSE_LEAVE` | GeoJson / Gml / Cluster |
| `GRAPHIC_ADDED` / `GRAPHIC_DELETED` | GraphicLayer |
| `PRECLUSTER` / `POSTCLUSTER` | ClusterLayer |
