# 分组 · 实例化 · 要素集 — GroupLayer · TransformableGroupLayer · InstanceLayer · FeatureCollection 系

入口 `map.layerManager`。

## GroupLayer（逻辑分组，`type:'group'`，本身不含数据）

- 方法：`addLayer(options)` `removeLayer(layer)` `removeAllChildren()`
- 成员：`show`（控制子图层整体显隐）

## TransformableGroupLayer（整体变换，集成拖拽 Gizmo，`type:'transformable'`）

```js
const tg = new Glodon.CIMCube.Layer.TransformableGroupLayer(
  { id: "planA", name: "方案A", children: ["b1", "b2"] },  // children: 图层 id 或图层
  map
);
map.layerManager.addLayer(tg);
tg.offset = [10, 0, 0];   // 读写
tg.rotation = [0, 0, Math.PI / 6];
tg.scale = 1.2;
```

- 成员：`show` `offset`(rw) `scale`(rw) `rotation` `rotationOrigin` `dragable` `dragCenter` `boundingVolume` `orientedBoundingBox` `boundingVolumeOrigin` `sizeForDragger` `showOrientedBoundingBox` `transformChanged`(CIMCubeEvent)
- 方法：`addLayer(layer)` `removeLayer(layer)` `removeAllChildren()` `getLayers()` `addDragHandle(options)` `getDragableTarget()` `setMap(map)` `flyTo()` `dispose()`
- 事件：`TRANSFORM_CHANGED`

## InstanceLayer / InstanceLayerModel（GPU 实例化，`type:'instance'`）

```js
const inst = new Glodon.CIMCube.Layer.InstanceLayer({
  id: "lamps", name: "路灯", origin: [116.41667, 39.91667, 500.0],
  show: true, phongLighting: false, useDefaultMetallicRoughness: true, doubleSided: undefined,
});
inst.addModel("./data/lamp.glb", "lamp-model");   // url, id
const model = inst.getModel("lamp-model");        // InstanceLayerModel
model.setInstanceData([{ position: [116.42, 39.91, 0] }, ...]);
model.add({ position: [116.43, 39.92, 0] });
```

- **InstanceLayer**：成员 `origin` `show` `doubleSided` `phongLighting` `enableRealisticRendering` `boundingSphere`；方法 `addModel(url, id)` `getModel(id)` `removeModel(id)` `getInstance(id)` `addDragHandle(options)` `flyTo()` `dispose()`
- **InstanceLayerModel**：构造 `{url, id, origin, layer}`；成员 `id` `url` `position` `show` `instanceData` `boundingSphere`；方法 `setInstanceData(data)` `add(data)` `get(id)` `remove(id)` `removeAll()` `getInstancePosition(ins)` `getModelInstance(id)`

## FeatureCollection 系（低层）

- **FeatureCollection**（低层基类）：由引擎内部构造（`new FeatureCollection(ctx, cacher)`，业务侧不要自行 `new`）；成员 `featureMap`(ro)；方法 `add(id, value)` `get(id)` `has(id)` `remove(id)`（图层隐藏或数据未加载时 `get`/`has` 为空/false）
- **GeoJsonFeatureCollection**：由引擎内部构造（`new GeoJsonFeatureCollection(ctx, cacher)`）；方法 `show()` `hide()` `getCenter()` `setHighLight(color?, displayOnTop?)` `disableHighLight()` `setFlash(opts?)` `setBreath(opts)` `disableBreath()` `animation(style)` `disableAnimation()` `setFeatureEffect(mode, options)` `clearFeatureEffect()`
- **FeatureStyleParser**：`new FeatureStyleParser()`；方法 `getStyle()` `getItemStyle(value)` `getKey()` `isInited()`
