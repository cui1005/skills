# 交互类图层 — GraphicLayer · ClusterLayer · DivLayer

入口 `map.layerManager`。

## GraphicLayer（`type:'graphic'`，承载 `cimcube-graphic` 的图元）

- 构造：`type`（绘制类型）`style` `config` `normalize`(默认 false)
- 成员：`show` `enableEdit` `allowEmptySelection`（默认 true）
- 方法：`addGraphic(g)` `removeGraphic(g)` `removeAllGraphics()` `getGraphicById(id)` `getAllGraphics()` `flyTo()` `dispose()` `startDraw(options?)` `stopDraw()` `clearDraw()` `startEditing(graphic)` `stopEditing()` `setCursor()` `setTipsMessage(msg)`
- 事件：`ENTITY_CLICK`（以及 `ENTITY_MOUSE_ENTER`/`ENTITY_MOUSE_LEAVE`）、`GRAPHIC_ADDED`/`GRAPHIC_DELETED`
- ⚠️ **没有** `add / update / removeById / getGraphicsByCondition`（臆造）

## ClusterLayer（点聚合，`type:'cluster'`）

```js
const cluster = new Glodon.CIMCube.Layer.ClusterLayer({
  id: "poi-cluster", name: "POI聚合",
  enabled: false, show: true,
  pixelRange: 20, minimumClusterSize: 5, cartesian3Range: 0,
  expandRadius: 20, expandRadiusAdd: 40,
  imageSize: [50, 59], labelSize: 20, labelColor: "#ffffff", labelOffset: [-4, -28],
  clampToGround: false, disableDepthTest: false,
  translucencyByDistance: [near, nearAlpha, far, farAlpha],
});
cluster.add(entity);            // 添加 Entity（billboard/point/label）
cluster.getById(id); cluster.removeById(id); cluster.remove(entity); cluster.removeAll();
```

- 方法：`add(options)` `remove(entity)` `removeById(id)` `removeAll()` `getById(id)` `contains(entity)` `flyTo()` `dispose()`
- 事件：`ENTITY_CLICK` `ENTITY_MOUSE_ENTER` `ENTITY_MOUSE_LEAVE` `PRECLUSTER` `POSTCLUSTER`

## DivLayer（HTML 覆盖层，`type:'div'`）

```js
const div = new Glodon.CIMCube.Layer.DivLayer({
  id: "div", name: "标注层",
  horizontalOrigin: Cesium.HorizontalOrigin.CENTER,
  verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
  enableThinning: false, thinningTolerance: 10,
  clampToGround: false, zIndex: 10, pointerEvent: true, mapEvent: false,
});
div.addDiv([lon, lat, height], domElement, offset);   // 添加 DIV
```

- 方法：`addDiv(coordinates, domDiv, offset)` `removeDiv(value)` `removeAllDiv()` `getDiv(divId)` `getAllDivs()` `query(filter)` `bindLayer(bindLayer, options)` `unbindLayer()` `setVisibleByDistance(near?, far?)` `flyTo()` `dispose()`
