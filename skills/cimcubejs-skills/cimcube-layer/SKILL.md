---
name: cimcube-layer
description: "CIMCube 图层体系 LayerManager：图层增删查改/显隐/排序/飞向/分组树；覆盖影像、地形、Tiles3D、GLTF、I3S、S3M、USD、矢量瓦片 MVT、GeoJSON、GML、KML、点聚合、DIV 覆盖、分组与实例化图层。当用户需要加载/移除/显隐/排序图层、切换底图地形、飞向图层、加载矢量与模型数据、做点聚合或 DIV 弹窗、按图层组批量管理时激活。"
---

# 图层体系 Layer

> **基线**：命名空间 `Glodon.CIMCube.Layer`（24 个类）
> **总入口**：`map.layerManager`（`Glodon.CIMCube.Layer.LayerManager`，CIMCube 实例只读成员）
> **文档与运行时口径**：本页参数/默认值/方法均已对照 25 份 API_DOC 类页 + `cimcube.js` 实现逐项核实
> **定位**：本 SKILL.md 只做**索引路由 + 跨域铁律**。各类图层完整构造选项、方法、事件与示例全部外置于同目录 `references/<方法族>.md`。**写某类图层的完整代码前，先读对应文件**，不要凭记忆重写示例。

## 一、三条铁律（最容易错）

1. **`map.addLayer(...)` 不存在**。主类 `CIMCube` 只暴露 `layerManager` 成员，没有 addLayer/removeLayer/getLayer 直通方法（API_DOC 示例里的 `map.addLayer(...)` 是文档笔误，官方 demo 用 `map.layerManager.addLayer(...)`）。
2. **图层显隐属性是 `show`（不是 `visible`）；透明度是 `alpha`（不是 `opacity`）**。
3. **图层事件必须传数值枚举** `Glodon.CIMCube.Global.Defines.LayerEventType.*`；传字符串 `"LOADED"` 不会触发（运行时事件 `type` 是数字）。

```js
// ★ 正确入口：一切图层操作走 map.layerManager
const LM = map.layerManager;

const tdt = new Glodon.CIMCube.Layer.ImageryLayer({
  id: "tdt-img",
  name: "天地图影像",
  imageryType: "tdt_img",                    // ★ 不是 type
  options: { minimumLevel: 1, maximumLevel: 18 },
});
LM.addLayer(tdt);                            // 也可 LM.addLayer({ type:Glodon.CIMCube.Global.Defines.LayerType.IMAGERY, ... })
```

## 二、跨域约定（适用于全部图层）

1. **`type` 与 `imageryType`/`terrainType` 分工**：`addLayer` 配置对象用 `type:Glodon.CIMCube.Global.Defines.LayerType.*`（如 `TILES3D:'3dtiles'`）分发；图层**构造选项**里影像用 `imageryType`、地形用 `terrainType`，**不写 `type`**。
2. **模型类变换三件套**：所有模型/瓦片类图层用 `origin`（`[lon,lat,height]` 经纬高原点）+ `offset`（`[东,北,上]` 米）+ `rotation`（`[绕X,绕Y,绕Z]`）+ `scale`，**不是** `position/heading/pitch/roll`。
3. **显隐 / 透明度 / 拾取**：`show`、`alpha`、`pickable`、`canSelected`、`canHighlight`、`selectedColor`（全表见 `references/base-layer.md`）。
4. **事件**：统一 `layer.on(Defines.LayerEventType.*, fn)`（返回取消函数）；取值表与各图层可用事件见 `references/events.md`。
5. **增删查改**：`addLayer` 幂等（同 id 返回已有图层）；`getLayer` 只接受 **String id 或图层实例**；`flyToLayer` 同样不接受 name。详见 `references/layer-manager.md`。

### 常用枚举（`Glodon.CIMCube.Global.Defines`）

| 枚举 | 用途 | 关键值 |
|---|---|---|
| `LayerType` | `addLayer` 配置对象的 `type` 字段 | `IMAGERY:'imagery'` `TERRAIN:'terrain'` `TILES3D:'3dtiles'` `GLTF:'gltf'` `GSMM:'gsmm'` `I3S:'i3s'` `S3M:'s3m'` `USD:'usd'` `VECTORTILE:'vectortile'` `GEOJSON:'geojson'` `KML:'kml'` `GML:'gml'` `DIV:'div'` `GRAPHIC:'graphic'` `CLUSTER:'cluster'` `GROUP:'group'` `WATER:'water'` `INSTANCE:'instance'` `TRANSFORMABLE:'transformable'` `FDB:'fdb'` |
| `ImageryLayerType` | `ImageryLayer.imageryType` | `XYZ:'xyz'` `WMS:'wms'` `WMTS:'wmts'` `TMS:'tms'` `TDT_IMG:'tdt_img'` `TDT_VEC/TDT_TER/TDT_CVA/TDT_CIA/TDT_CTA` `GAODE_IMG/GAODE_VEC/GAODE_CIA` `BAIDU_IMG/BAIDU_VEC` `ARCGIS_IMAGE/ARCGIS_IMG/ARCGIS_GRAY` `TENCENT_IMG` `GOOGLE_IMG` `SUPERMAP_IMAGE` `OFFLINE_IMG/OFFLINE_MIX` `CUSTOM` |
| `TerrainLayerType` | `TerrainLayer.terrainType` | `DEFAULT` 等 |
| `ShadowMode` | 阴影模式 | `DISABLED:'disabled'` `ENABLED:'enabled'` `CAST_ONLY:'cast_only'` `RECEIVE_ONLY:'receive_only'` |
| `ClassificationType` | 分类类型 | `TERRAIN` `TILES3D` `BOTH` |
| `FeatureRenderMode` | GeoJson/Gml 渲染模式 | `ENTITY` `PRIMITIVE` `BUFFER` |
| `FeatureEffectMode` | 要素特效 | `HIGHLIGHT` `FLASH` `ANIMATION` `BREATH` |
| `LayerEventType` | 图层事件（数值枚举） | 见 `references/events.md` |

## 三、24 类路由

| 类 | `type` 值 | 用途 | 写码前必读 |
|---|---|---|---|
| `LayerManager` | — | 总管理器（增删查改/显隐/排序/飞向/分组挂载） | `references/layer-manager.md` |
| `BaseLayer` | — | 所有图层父类：公共构造选项 / 成员 / 方法 / 事件订阅 | `references/base-layer.md` |
| `ImageryLayer` | `imagery` | 影像底图（XYZ/WMS/WMTS/TMS/天地图/高德/百度/ArcGIS/腾讯/离线） | `references/imagery-terrain.md` |
| `TerrainLayer` | `terrain` | 地形（QuantizedMesh/Heightmap/GeoServer/超图） | `references/imagery-terrain.md` |
| `Tiles3DLayer` | `3dtiles` | 3D Tiles（b3dm/i3dm/pnts/glb）——最常用、功能最多 | `references/tiles3d.md` |
| `GltfLayer` | `gltf` | 单体 glTF/GLB | `references/model-layers.md` |
| `GSMMLayer` | `gsmm` | 广联达 GSMM（.gsmm + assets） | `references/model-layers.md` |
| `I3STilesLayer` | `i3s` | Esri I3S | `references/model-layers.md` |
| `S3MTilesLayer` | `s3m` | 国产 S3M 倾斜 | `references/model-layers.md` |
| `USDLayer` | `usd` | USD（.usda/.usdc/.usd/.usdz） | `references/model-layers.md` |
| `VectorTileLayer` | `vectortile` | MVT/PBF 矢量瓦片（Mapbox 风格） | `references/vector-layers.md` |
| `GeoJsonLayer` | `geojson` | GeoJSON 点/线/面 | `references/vector-layers.md` |
| `GmlLayer` | `gml` | OGC GML | `references/vector-layers.md` |
| `KmlLayer` | `kml` | KML/KMZ | `references/vector-layers.md` |
| `GraphicLayer` | `graphic` | 图元层（承载 cimcube-graphic 的 Graphic） | `references/interaction-layers.md` |
| `ClusterLayer` | `cluster` | 点聚合 | `references/interaction-layers.md` |
| `DivLayer` | `div` | HTML DIV 覆盖层（弹窗/气泡） | `references/interaction-layers.md` |
| `GroupLayer` | `group` | 逻辑分组 | `references/group-instance.md` |
| `TransformableGroupLayer` | `transformable` | 可变换分组（整体平移/旋转/缩放） | `references/group-instance.md` |
| `InstanceLayer` | `instance` | GPU 实例化层 | `references/group-instance.md` |
| `InstanceLayerModel` | — | 实例模板（配合 InstanceLayer） | `references/group-instance.md` |
| `FeatureCollection` | — | 要素集基类（低层） | `references/group-instance.md` |
| `GeoJsonFeatureCollection` | — | GeoJSON 要素批量特效管理 | `references/group-instance.md` |
| `FeatureStyleParser` | — | 要素样式解析器 | `references/group-instance.md` |
| 事件系统（全量取值 + 各图层可用事件） | — | `Defines.LayerEventType` 数值表 | `references/events.md` |
| 常见模式 / 误用对照 | — | 6 个可套用配方 / 14 条坑 | `references/patterns.md` · `references/pitfalls.md` |

> 另：运行时还存在 **`WaterGeoJsonLayer`**（`type:'water'`，水体面图层），代码内 `createLayer` 支持但 API_DOC 未单独列页。

## See Also

- **cimcube-setup** — CIMCube 构造、`center` 初始视角、`map.layerManager` 的创建时机
- **cimcube-camera** — `map.camera`（`flyTo` 目标支持图层 → `layer.getBoundingSphere()`；`map.camera.flyTo({target: layer})`）
- **cimcube-graphic** — GraphicLayer 承载的 14 个 Graphic 图元、样式与绘制/编辑
- **cimcube-widgets** — 图层树面板（初始化时 `params: { layerControl: true }` 开启，面板由 CIMCube 自动挂载）、`MapLegendWidget` 图例（cimcube-widgets/references/MapLegend.md）
- **cimcube-utils** — `MapSplit`/`MultiViewPort` 分屏、`SpatialTilesetCloner` 3DTiles 克隆（cimcube-utils/references/MapSplit.md、cimcube-utils/references/SpatialTilesetCloner.md）
- **cimcube-scene** — 阴影/天空盒/反射盒/等场景级渲染设置（与图层光照属性联动）
- **cimcube-realistic-rendering** — 真实感管线与光源（PBR 材质反射配合 `map.scene.reflectBox`）
- **cimcube-physics** — 场景物理引擎（`setCollisionModel` 使图层参与刚体碰撞）
- **cimcube-global** — `Global.Defines` 全量枚举（LayerType/ImageryLayerType/LayerEventType/ShadowMode 等）

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
