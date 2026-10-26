# 矢量类图层 — GeoJsonLayer · GmlLayer · KmlLayer · VectorTileLayer

入口 `map.layerManager`。

## GeoJsonLayer（`type:'geojson'`）

```js
const geo = new Glodon.CIMCube.Layer.GeoJsonLayer({
  id: "poi", name: "兴趣点",
  data: geojsonObject,                    // 或后续 load(data)
  mode:Glodon.CIMCube.Global.Defines.FeatureRenderMode.ENTITY,
  style: styleObjectOrUrl,                // 参考官方样式规范（user-guide/405）
  height: 0, extrudedHeight: 0, blockHeight: 0,
  defaultColor: "#FFFF00", defaultImage: "", defaultSize: [30, 36],
  clampToGround: false, clampToModel: false,
  enableLighting: true, zIndex: 0, displayOnTop: false,
  enableThinning: false, thinningTolerance: 10,
  enableCluster: false, cluster: {...},
});
```

- 方法：`load(data)` `addFeatures(features)` `removeAll()` `queryFeatures(filter)` `getFeatures()` `getFields()` `getGeometryType()` `getBoundingSphere()` `flyTo()` `dispose()`
- 样式/高度：`setStyle(style)` `getStyle()` `setEntityStyle(entity, style)` `setHeight(h)` `setHeightByField(field, factor?)` `setBlockByField(field, factor?)` `setBlockByValue(blockHeight)`
- 标注：`setLabelStyle(options, callback?)` `getLabelStyle()` `setLabelHeight(h)` `setLabelVisibleByDistance(near?, far?)` `setLabelAlphaByDistance(near?, nearValue?, far?, farValue?)` `setLabelOffsetByDistance(...)` `setLabelScaleByDistance(...)`
- 距离显隐/图例：`setVisibleByDistance(near?, far?)` `getLegendInfo()`
- 扩散特效：`enableSpreadEffect(options)`（配构造键 `spreadCenter{lon,lat,height}` `spreadTime` `spreadRepeat` `spreadGradual`）
- 自有事件：`ENTITY_CLICK` `ENTITY_MOUSE_ENTER` `ENTITY_MOUSE_LEAVE` `LOADED`

## GmlLayer（`type:'gml'`）

构造选项与 GeoJsonLayer 基本一致；方法为 `getEntities()` `getEntityByID(id)` `getEntitiesByProperty()` `getEntityProperty(entity, prop)` `getEntityPropertyList(entity)` + 与 GeoJson 同名的样式/高度/标注/距离方法。自有事件同 GeoJsonLayer。

## KmlLayer（`type:'kml'`）

构造仅 `show`（默认 true）、`clampToGround`（默认 false）；方法 `getEntities()` `flyTo()` `dispose()`。

## VectorTileLayer 矢量瓦片 MVT/PBF（`type:'vectortile'`）

```js
const mvt = new Glodon.CIMCube.Layer.VectorTileLayer({
  id: "mvt", name: "矢量瓦片",
  url: "http://.../style.json",   // 与 style 二选一
  // style: {...},                 // Mapbox style-spec JSON 对象
  bounds: [-180, -85.051129, 180, 85.051129],
  height: 0, clampToGround: false,   // clampToGround 暂不支持
});
```

- 成员：`show` `height` `offset` `classificationType` `clampToGround`
- 方法：`addSource(id, source)` `removeSource(id)` `getSource(id)` `addStyleLayer(layerObject, before?, options?)` `removeStyleLayer(id)` `getStyleLayer(id)` `hasStyleLayer(id)` `moveStyleLayer(id, before?)` `getStyle()` `setFilter(id, filter, options?)` `getFilter(id)` `getLayoutProperty(id, name)` `getPaintProperty(id, name)` `queryRenderedFeatures(geometry?, options?)` `setFeatureState(feature, state)` `getFeatureState(feature)` `removeFeatureState(feature, key)` `flyTo()` `dispose()`
