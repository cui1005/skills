# 误用对照表（含运行时实证）

入口 `map.layerManager`。

| ❌ 易错写法 / 常见误解 | ✅ 正确口径 | 依据 |
|---|---|---|
| `map.addLayer({type, url})` | **`map.layerManager.addLayer(...)`**（主类无 addLayer） | 运行时主类方法扫描 + 官方 demo |
| `LM.getLayerById(id)` / `getLayerByName` / `getLayerByIndex` | **`LM.getLayer(id)`**（只接受 String 或图层实例） | 运行时（这些方法不存在） |
| `LM.getVisibleLayers()` | 遍历 `getAllLayers()` 按 `show` 过滤（无此方法） | 运行时 |
| `LM.raise / lower / moveTo / swap` | **`LM.moveLayer(value, target?, index?)`** | 运行时 |
| `layer.visible = false` | **`layer.show = false`** | BaseLayer 实现（无 visible） |
| `layer.opacity = 0.5` | **`layer.alpha = 0.5`** | BaseLayer |
| `ImageryLayer({ type: 'xyz' })` | **`imageryType: 'xyz'`**（`type` 用于 addLayer 分发的 `'imagery'`） | ImageryLayer 构造选项 |
| `GltfLayer({ position, heading, pitch, roll })` | **`origin / rotation / offset / scale`** | GltfLayer 构造选项 |
| `layer.on("LOADED", fn)` | **`layer.on(Defines.LayerEventType.LOADED, fn)`**（数值枚举） | 运行时 `fire({type: 数字})` |
| `GraphicLayer.add / update / removeById / getGraphicsByCondition` | `addGraphic / removeGraphic / removeAllGraphics / getGraphicById / getAllGraphics` | GraphicLayer 页 |
| `getAllLayers()` 能拿到全部 | 会**过滤 `hiddenInLayerTree:true`** 的图层（`getLayer` 仍可取） | BaseLayer 文档 |
| `getTerrainLayer()` | 已 Deprecated，用 **`getTerrainLayers()`** | LayerManager 页 |
| `LM.flyToLayer("名称")` | 只接受 **id 或图层实例** | 运行时 `getLayer(e)?.flyTo()` |
| `LayerType.TILES3D === 'tiles3d'` | 字符串值是 **`'3dtiles'`** | 枚举定义 |
