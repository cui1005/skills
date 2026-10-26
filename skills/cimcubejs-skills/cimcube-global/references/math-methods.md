# Global.Math — 真实静态方法全表（runtime 25 个）

> 本文件对应 SKILL.md 路由表的《25 个静态方法：单位与颜色 / 经纬度↔笛卡尔 / 图层局部坐标与矩阵（含 SVD 配准）/ 点与元素范围判定 / AABB·OBB / 贝塞尔插值》行。

> 别名建议：`const C = Glodon.CIMCube.Global.Math;`，注意别与浏览器 `Math` 混淆。
> 特点：本模块是 **Cesium 坐标系 / 图层坐标 ↔ 世界坐标的“转换与判定”工具箱**，不是通用数学库；通用距离/面积请用 `cimcube-utils` 的 `Utils.Measure`（`cimcube-utils/references/Measure.md`）。

## 2.1 单位与颜色

| 方法 | 参数 → 返回 | 说明 |
|---|---|---|
| `toRadians(degrees)` | 度数 → 弧度 | 度 → 弧度 |
| `toDegrees(radians)` | 弧度 → 度数 | 弧度 → 度 |
| `toCesiumColor(color)` | `#FFFFFF\|rgba()\|red\|Cesium.Color` → `Cesium.Color` | 任意色值字符串转 Cesium 颜色 |

## 2.2 经纬度 ↔ 世界/笛卡尔坐标（★最常用）

| 方法 | 参数 → 返回 | 说明 |
|---|---|---|
| `llhToCartesian(lon, lat, height)` | (Number, Number, Number) → `{x,y,z}` | 经纬度+高(米) → 世界笛卡尔(ECEF) |
| `cartesianToLlh(p)` | `{x,y,z}` → `{lon,lat,height}` | 世界笛卡尔 → 经纬度 |
| `cartesianToCoordinates(cartesian)` | `Cartesian3` → `[lon,lat,height]` | Cartesian3 → 坐标数组 |
| `coordinatesToCartesian(coordinate)` | `[lon,lat,height]` → `Cartesian3` | 坐标数组 → Cartesian3 |
| `cartesianToCartographic(cartesian)` | `Cartesian3` → `Cartographic` | → 地理坐标对象（可再做经纬度弧度换算） |
| `cartographicToCartesian(cartographic)` | `Cartographic` → `Cartesian3` | 反向 |
| `cartesianArrayToCoordinatesArray(cartesianArray)` | `Cartesian3[]` → `[[lon,lat,height],…]` | 批量转换 |
| `coordinatesArrayToCartesianArray(coordinatesArray)` | `[[lon,lat,height],…]` → `Cartesian3[]` | 批量转换 |
| `modelToWorldCoordinates(coordinates, transform)` | 单个扁平数组 `[x,y,z]` + `Matrix4` → 单个 `[lon,lat,height]` | 模型坐标经变换转经纬度。⚠️ 实测入参是**单点扁平数组**（内部 `Cartesian3.fromArray`，传 `{x,y,z}` 对象或嵌套数组会得到 `[null,null,null]`），返回也是**单点**而非数组 |

```js
// 拿一个 S3M 瓦片的包围球中心：Cartesian3 → 经纬度 → 让相机飞过去（配合 cimcube-camera）
const cart = layer.boundingSphere?.center;                 // Cartesian3
const llh  = M.cartesianToLlh(cart);                      // {lon,lat,height}
map.camera.flyTo({ destination: { lon: llh.lon, lat: llh.lat, height: 2000 } });
```

## 2.3 图层局部坐标 ↔ 世界坐标 / 变换

| 方法 | 参数 → 返回 | 说明 |
|---|---|---|
| `getWorldPositionByLocalPosition(projectPosition, localPosition, tiles3DLayerPosition)` | 3×`Cartesian3` → `Cartesian3` | 以指定图层原点为基准：项目原点 + 局部坐标 → 世界坐标 |
| `coordinateTranslate(position, transformationMatrix)` | `Cartesian3` + `Matrix4` → `Cartesian3` | 矩阵变换坐标 |
| `caluateMatrix(localPosition, sourceCoordinate, targetCoordinate)` | 原点 `{x,y,z}` + 源/目标坐标系 → `Matrix4` | 构造三维坐标转换矩阵；坐标系取 `"local"`（局部）或 `"EPSG:4978"`（三维地理） |
| `computeSVDMatrix(sourcePoints, targetPoints)` | 两组坐标数组 → `Matrix4` | 三维 SVD 配准矩阵 |
| `computeSVDMatrix2D(sourcePoints, targetPoints)` | 图纸点 `[{x,y}]` + 目标 `[{lon,lat}]` 或 `Cartesian3[]`（≥3 对）→ `Matrix4` | 二维拟合（图纸坐标 → 三维世界坐标，椭球切平面相似变换：旋转+平移+缩放） |
| `offsetToLongLat(llh, offset)` | `{lon,lat}` + `{x,y}` → `{lon,lat}` | 二维平面近似偏移（文档注明有误差） |

```js
// 图纸坐标配准：≥3 对控制点 → 求得 Matrix4 → 任意图纸点变世界坐标
const m4 = M.computeSVDMatrix2D(
  [{x: 0, y: 0}, {x: 1000, y: 0}, {x: 0, y: 1000}],
  [{lon: 116.397, lat: 39.909}, {lon: 116.405, lat: 39.909}, {lon: 116.397, lat: 39.915}]
);
// const worldPos = Glodon.CIMCube.Cesium.Matrix4.multiplyByPoint(m4, new Cartesian3(x, y, 0), new Cartesian3());
```

## 2.4 点 / 元素 范围判定

> `element` 接受 **OBB**（`OrientedBoundingBox`，含 `halfAxes`）或 **AABB**（`{minimum:{x,y,z}, maximum:{x,y,z}}`）。

| 方法 | 参数 → 返回 | 说明 |
|---|---|---|
| `calculatePointInCircleRange(center, minRadius, maxRadius, point)` | 圆心 `{lon,lat}` + 半径(m) + 点 `{lon,lat}` → `boolean` | 点是否在圆环内；圆时 `minRadius=0` |
| `calculateElementInCircleRange(center, minRadius, maxRadius, element)` | … → `Cesium.Intersect` | 包围盒与圆环关系：`OUTSIDE:-1 / INTERSECTING:0 / INSIDE:1` |
| `calculatePointInAndOnPolygon(point, polygon)` | 点 `{lon,lat}` + 环 `[{lon,lat},…]` → `boolean` | 点在内**或边界上** |
| `calculateElementInPolygonRange(polygon, element)` | 环 + OBB/AABB → `Cesium.Intersect` | 包围盒与多边形关系 |

## 2.5 AABB / OBB 包围盒

| 方法 | 参数 → 返回 | 说明 |
|---|---|---|
| `convertLocalAabbToWorldObb(projectPosition, aabb)` | 项目原点 `Cartesian3` + AABB → `OrientedBoundingBox` | 局部 AABB → 世界 OBB |
| `transformAndRotateAABB(layerTransform, toLocalMatrix, boundingBox [, outBoundingBox])` | 局部→世界 `Matrix4` + 世界→目标 `Matrix4` + AABB → AABB | 模型 AABB 过两个矩阵变换并重算 AABB |

## 2.6 曲线插值

| 方法 | 参数 → 返回 | 说明 |
|---|---|---|
| `roundingByBezier(list, interpolationCount=15, approximation=0)` | 坐标数组 + 插值个数 + 逼近值(0~1) → 坐标数组 | 贝塞尔圆滑路径点（`approximation=0` 过支点，`1` 穿支点，不宜过多） |

## 2.7 方法级黑名单（已统一抽到 pitfalls.md）

> ⚠️ APIDOC 列出但 runtime 不导出的 Math 方法黑名单与 `caluateMatrix` 拼写差异，已统一抽到 `references/pitfalls.md`。写码前先读该节，本文件不再重复维护以免漂移。
