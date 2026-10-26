# Camera 属性成员 · 事件 · dispose

入口 `map.camera`。

## 成员（属性）速查

| 成员 | 类型 | 读写 | 说明 |
|---|---|---|---|
| `position` | `Cartesian3` | 只读 getter | 相机世界坐标（ECEF） |
| `positionCartographic` | `Cartographic` | 读写 | 相机经纬高（弧度制经纬度，Cesium Cartographic） |
| `heading` / `pitch` / `roll` | `Number` | **只读** getter | 当前姿态，弧度。0 航向=北、逆时针为正（东=+π/2）；pitch 负值向下（-π/2 正俯视）；roll 一般 0。**没有 setter**——想改姿态用 `setView`/`flyTo`/`lookAt` 的 orientation |
| `fov` | `Number` | 读写 | 视野角（**度**）。透视相机才有效；非透视 frustum 时 getter 返回 `undefined`，setter 静默忽略 |
| `mvtZoom` | `Number` | 只读 | 当前地图层级（仅矢量切片图层场景有含义） |
| `decreaseRequestingWhileFlying` | `Boolean` | 写（默认 false） | 开关：开启后 `flyTo` 才透传 `decreaseRequesting`，飞行中减少瓦片请求 |
| `trackedModel` | `GltfLayer \| undefined` | 读写 | 跟踪的模型图层：赋值后相机自动看向该 gltf 模型包围球（模型 ready 后 lookAt 其包围球中心） |
| `changed` / `moveStart` / `moveEnd` | `Cesium.Event` | 只读 | 相机事件，见下方 |

> ⚠️ **不存在** `camera.x / y / z` 属性（易错写法）。ECEF 坐标只在 `getCameraInfo().x/y/z` 与 `getCurrentViewInfo().positionWC.{x,y,z}` 中出现。

## 跟踪模型 / fov / mvtZoom 用法

```js
// trackedModel：赋值后相机看向该模型（gltf 图层 ready 后按其包围球 lookAt）
const gltfLayer = map.layerManager.getLayer("my-gltf");
map.camera.trackedModel = gltfLayer;          // 开始跟踪
map.camera.trackedModel = undefined;          // 取消（回到自由视角）
// 视野角：fov 单位为【度】，读写
console.log(map.camera.fov);                  // 如 60
map.camera.fov = 45;
// mvtZoom：矢量切片图层下的当前层级（只读）
console.log(map.camera.mvtZoom);
```

## 相机事件 `changed` / `moveStart` / `moveEnd`

三个成员是**内核 Event 对象**（不是 map 上的 `on("…")` 事件名），用 `addEventListener` 订阅：

```js
const onChanged = () => console.log("camera position changed");
map.camera.changed.addEventListener(onChanged);          // 相机位置发生改变
map.camera.moveStart.addEventListener(() => console.log("move start"));  // 开始移动/飞行
map.camera.moveEnd.addEventListener(() => console.log("move end"));      // 停止移动/飞行
// 取消订阅：map.camera.changed.removeEventListener(onChanged)
```

常用：`moveEnd` 监听飞行结束做后续联动；`changed` 做视角状态条/URL 同步（注意高频，自行节流）。

## dispose

```js
map.camera.dispose();   // 停止自转 + cancelFlight，释放相机（一般随 map.dispose() 一并处理）
```
