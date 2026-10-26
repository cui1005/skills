# LayerManager — 图层总管理器（入口）

入口 `map.layerManager`（`Glodon.CIMCube.Layer.LayerManager`，CIMCube 实例只读成员）。

## 添加

| 方法 | 说明 |
|---|---|
| `addLayer(opts, parent?)` | 添加图层。`opts` 可以是 **图层实例** 或 **配置对象**（必填 `type`，其余键与对应图层构造选项一致）。`parent` 为 `GroupLayer` 实例或分组 id，**缺省挂到根分组**（根分组 id `cimcube_root`、name `场景`）。返回图层实例 |
| `addSceneLayer(options, parentLayer?)` | 添加场景图层（需 `options.type`） |
| `addSceneLayers(options[], parentLayer?)` | 批量添加场景图层 |

```js
const LM = map.layerManager;
// ① 传配置对象（自动 new 对应图层类）
const tiles = LM.addLayer({
  type: Glodon.CIMCube.Global.Defines.LayerType.TILES3D, // '3dtiles'
  id: "bim-tiles",
  name: "BIM 模型",
  url: "./data/tileset.json",
  maximumScreenSpaceError: 16,
});
// ② 传图层实例
const gltf = LM.addLayer(new Glodon.CIMCube.Layer.GltfLayer({
  id: "car", name: "小车", url: "./data/car.glb",
  origin: [116.39, 39.90, 0],
}));
// ③ 挂到分组
const group = LM.addLayer({ type:Glodon.CIMCube.Global.Defines.LayerType.GROUP, id: "g1", name: "建筑组" });
LM.addLayer({ type:Glodon.CIMCube.Global.Defines.LayerType.TILES3D, id: "b1", name: "A栋", url: "a.json" }, "g1");
```

> **运行时定论**：`addLayer` 幂等——若 `opts.id` 已存在，会 `console.warn("add duplicated layer! id: ...")` 并**返回已有图层**（不会重复添加）；`opts` 为空时告警并返回。`createLayer` 遇到未知 `type` 告警返回 `undefined`。

## 查询

| 方法 | 说明 |
|---|---|
| `getLayer(value)` | 按 **id（字符串）** 或 **图层实例** 取图层；其他类型抛错（⚠️ **不接受数字**）。（部分资料写作 `getLayerById`） |
| `getLayerMap()` | 返回 `Map<id, layer>` |
| `getAllLayers()` | 所有图层（**不含** `hiddenInLayerTree:true` 的图层） |
| `getLayersByType(type)` | 按 `LayerType` 取图层（同样过滤 `hiddenInLayerTree`） |
| `hasLayer(value)` | 是否存在（接受 String / Number / BaseLayer） |
| `getImageryLayers()` | 所有影像图层 |
| `getTerrainLayers()` | 所有地形图层（`getTerrainLayer()` 已 **Deprecated**） |
| `get3DTilesLayers()` | 所有 3D Tiles 图层 |
| `getGltfLayers()` | 所有 glTF 图层 |
| `getGSMMLayers()` | 所有 GSMM 图层 |
| `getI3SLayers()` | 所有 I3S 图层 |
| `getS3MLayers()` | 所有 S3M 图层 |
| `getGraphicLayers()` | 所有图元图层 |

```js
const layer = LM.getLayer("bim-tiles");
const all   = LM.getAllLayers();
const tiles3d = LM.get3DTilesLayers();
```

## 移除 / 显隐 / 顺序 / 重命名

| 方法 | 说明 |
|---|---|
| `removeLayer(value, isDispose=false)` | 移除单个图层 |
| `removeLayers(layerArray, isDispose=false)` | 批量移除 |
| `removeLayersByType(type, isDispose=false)` | 按类型移除 |
| `removeAllLayers(isDispose=false)` | 移除全部 |
| `showLayer(layer, show=true)` | 单图层显隐 |
| `showLayers(layerArray, show=true)` | 批量显隐（`layerArray` 必须是数组，否则抛错） |
| `moveLayer(value, target?, index?)` | 移动图层（目标分组 / 位置） |
| `renameLayer(value, name)` | 重命名 |
| `flyToLayer(layer)` | 相机飞向图层，内部 `getLayer(layer)?.flyTo()`（⚠️ 只接受 **id 或图层实例**，不支持按 name） |
| `dispose()` | 释放图层管理器（通常随 `map.dispose()`） |

> `isDispose` 为 `true` 时连同底层资源一起销毁；仅移除显示用默认 `false`。
