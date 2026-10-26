# Tiles3DLayer — 3D Tiles（最常用、功能最多）

入口 `map.layerManager`，`type: '3dtiles'`。

## 构造选项

**变换三件套（所有模型类图层通用，⚠️ 不是 position/heading/pitch/roll）**：

| 键 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `origin` | `[lon,lat,height]` | — | 图层原点（经纬高） |
| `offset` | `[东,北,上]` 米 | — | 平移偏移 |
| `rotation` | `[绕X,绕Y,绕Z]` | — | 旋转（右手系，默认绕包围盒中心） |
| `scale` | Number | `1.0` | 缩放 |
| `crs` | String | — | 数据坐标系，如 `"EPSG:4528"` |
| `projectTo2D` | Boolean | `false` | 精准投影到二维视图 |
| `maximumScreenSpaceError` | Number | `16` | 屏幕空间误差（LOD 精度），默认不修改，根据用户需求调整|
| `cacheable` | Boolean | `false` | 是否用 IndexedDB 缓存 |
| `shadowMode` | `ShadowMode` | — | 阴影模式 |
| `classificationType` | `ClassificationType` | — | 分类类型 |
| `enableSceneLighting` | Boolean | `true` | 是否受场景光照 |
| `enableCollision` | Boolean | `false` | 碰撞/贴瓦片高度 |
| `phongLighting` | Boolean | `false` | 冯氏光照 |
| `faceToCamera` | Boolean | `false` | 自动朝向相机；植被/树木图层必须开启，树木面片才会始终朝向相机 |
| `preferLeaves` / `skipLevelOfDetail` | Boolean | `false` | LOD 优化开关 |
| `metallicRoughnessFactor` | `[m,r]` | `[0.5,0.5]` | 金属度/粗糙度（白膜常用） |
| `doubleSided` | Boolean | `undefined` | 双面材质 |
| `outlineColor` | Color | `'#000000'` | 轮廓线（需 CESIUM_primitive_outline 扩展） |
| `pointCloudShading` | Object | — | 点云参数（`attenuation`/`geometricErrorScale`/`eyeDomeLighting`/`pointSize` 等） |
| `gradient` / `gradientImage` / `halo` / `breathe` / `dynamicPoint` / `topGlow` / `region` | Object | — | 各类特效参数（配 `set*/enable*` 方法） |

## 关键方法

| 分类 | 方法 |
|---|---|
| 定位 | `flyTo()` · `flyToSubLayer(layer, options?)` · `zoomToSubLayer(layer)` · `setStandardView(view, options?)` |
| 包围体 | `getBoundingSphere()` · `getBoundingBoxAABB()` · `getBoundingVolumeHeight()` · `getBoundingMaxHeight()` · `getPositionCenter()` · `getOriginWorldCoord()` |
| 坐标转换 | `getENUPosition(llh)` · `enuToWorldPosition(enu)` · `localToWorldPosition(local)` · `worldPositionToLocal(w)` · `worldPositionToLayerENU(w)` |
| 裁剪 | `setClippingPlanes(value, areasInfo?)` · `setMultiClippingPlanes(value, areasInfo?)` · `addMultiClippingPlane(id, collections, region)` · `removeMultiClippingPlane(id)` · `getClipTarget()` |
| 变换 | `move(angle, distance, mode?)`（**逆时针**） · `updateLayerTransformation()` · `spatialAdjust(sourcePoints, targetPoints)` |
| 单体要素 | `setFeatureEffect(ids, opts)` · `enableFeatureEffect()` · `disableFeatureEffect()` · `clearFeatureEffect()` · `removeFeatureEffect(ids, field)` · `setModelInteroperation(m)` · `setModelSilhouette(feature, opts)` · `getAllFeatureValues(field)` · `getFeaturesBoundingBox(ids, field, boxType)` · `disposeModelInteroperationForDrag()` |
| 特效（**两步：先 set* 设参数，再 enable* 开启**） | `setGradientEffect/enableGradientEffect` · `setGradientImageEffect/enableGradientImageEffect` · `setBreatheEffect/enableBreatheEffect` · `setDynamicHaloEffect/enableDynamicHaloEffect` · `setDynamicPointEffect/enableDynamicPointEffect` · `setLineamentEffect/enableLineamentEffect` |
| 子图层 | `showSubLayers(layers)` · `hideSubLayers(layers)` |
| 样式/调试 | `setStyleFromGeojson(style)` · `setColorBlendMode(options)` · `setDebugWireframe(value)` |
| 模型编辑/拖拽 | `addDragHandle(options)` · `removeDragHandle()` |
| 销毁 | `dispose()` |

- 只读成员：`tileset`（封装的内核 `Cesium3DTileset`）· `drawCalls` · `numberOfVertices` · `style` · `pointCloudStyle` · `pointSize` · `maxVisibleDistance` · `showBoundingVolume` · `showGeometricError` · `showUrl`
- 自有事件：`LOADED` `TILELOADED` `TRANSFORM_CHANGED` `VISIBLE`（见 `references/events.md`）

```js
// 加载 + 调整 + 特效 + 飞向
const t = LM.addLayer({
  type:Glodon.CIMCube.Global.Defines.LayerType.TILES3D, id: "bim", name: "BIM",
  url: "https://cimcube.glodon.com/Internal/CIMServer/GeoServices/v1/3DTiles/3335258249472608/tileset.json", // 示例数据，替换为你的 tileset 地址（本地文件或 http(s) URL）
  maximumScreenSpaceError: 8,
  origin: [116.39, 39.90, 0],
  rotation: [0, 0, Math.PI / 4],
});
t.setGradientEffect({ show: true, startHeight: 0, endHeight: 80, startColor: "#000000", endColor: "#00A0FF" });
t.enableGradientEffect(true);
LM.flyToLayer("bim");
```

> ⚠️ **`url` 必须是真实可达的 tileset 地址**（本地文件或 http(s) URL）。若写成 `./data/tileset.json` 这类本地占位路径，运行前需替换，否则会 404 且加载失败。

> **植被图层经验**：树木/植被类 3D Tiles 构造时必须加 `faceToCamera: true`，否则树木面片不随视角旋转（侧看变"纸片"）。建筑层不要开该属性——若建筑模型出现 billboard 化变形，说明 `faceToCamera` 开到了建筑层，应把植被与建筑拆成两个图层分别设置。
