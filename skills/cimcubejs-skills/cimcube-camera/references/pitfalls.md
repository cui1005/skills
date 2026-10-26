# 误用对照表（含运行时实证）

入口 `map.camera`。

| ❌ 易错写法 / 常见误解 | ✅ 正确口径 | 依据 |
|---|---|---|
| `camera.flyTo(target, opts)` 双参 | `camera.flyTo({target, …})` 单对象 | APIDOC 签名 + 运行时 |
| `easingFunction: "CubicInOut"` 或 Cesium 枚举 `EasingFunction.CUBIC_IN_OUT` | 只接受**大写下划线字符串白名单**（`"CUBIC_IN_OUT"`、`"SINUSOIDAL_IN_OUT"`、`"LINEAR_NONE"` 等 28 个）；**`LINEAR` 与 `QUADRATIC_*` 4 个查表缺失会被拒**；Cesium.EasingFunction 枚举成员一律非法。非法值行为：log `"unknown easingFunction!"` 且 **`throw undefined`**（非 Error）——`catch(e){ e.message }` 会二次 TypeError，必须 `String(e)` 兜底 | 运行时逐一实测 32 个名字 |
| `lookAt` 后只调 `setView` 解除锁定 | `setView`/`flyTo` **不**清零 `camera.transform`；锁定仍在 → 左键绕目标转、右键平移禁用的状态不会恢复。必须 `map.camera._camera.lookAtTransform(Matrix4.IDENTITY)` 先清零参考系，再 `setView` 复位。注：`map.camera` 包装层**未暴露** `lookAtTransform` | 运行时实测（lookAt 后 transform 非单位阵，`_camera.lookAtTransform(IDENTITY)` 才归位） |
| `map.camera.lookAtTransform` 存在可调用 | 包装层未暴露该方法；真实入口是底层 Cesium 相机 `map.camera._camera.lookAtTransform`，且 `map.scene.camera` 亦不存在 | 运行时成员扫描 + 实测 |
| `flyTo` target 只认图层 | 还支持 `{lon,lat,height}`、`Cartesian3`、`{x,y,z}`；图层还含 Div/I3S/S3M/GSMM。**Rectangle 仅 `flyToPoint:true` 可靠**（见下行） | 运行时 target 分支 |
| `flyTo({target: Rectangle})` 默认（包围球）模式 | ❌ 包围球分支不解析 Rectangle，实测飞到**北极点（lon 0 / lat 90 / 地下高度）**；✅ 加 `flyToPoint: true`（落矩形中心上空）或改用 `flyToBoundingSphere({bbox:[lon,lat,h,lon,lat,h]})`。注意 `setView` 的 Rectangle destination **正常**，不受此坑影响 | 运行时实测 |
| `flyToPoint` 用于图层整体概览 | `flyToPoint:true` 是"飞到点目标"；图层概览用不带 flyToPoint 的包围球模式 | 运行时分支 |
| 传 `{x,y,z}` 表达经纬度 | `{x,y,z}` 按 ECEF Cartesian3 原样透传，会飞错位；经纬度一律 `lon/lat/height` | 运行时转换分支 |
| `camera.x/y/z` 属性只读可用 | 不存在；ECEF 只在 `getCameraInfo()`/`getCurrentViewInfo().positionWC` 结果里 | 成员扫描 |
| `heading/pitch/roll` 可写 | **只读**；改姿态走 `setView/flyTo/lookAt` 的 orientation | 访问器扫描（仅 getter） |
| `fov` 是弧度 | get/set 均为**度** | 访问器扫描（toDegrees/toRadians） |
| `flyToBoundingSphere(sphere, opts)` | `flyToBoundingSphere(options)`；center+radius 或 bbox；offset.heading/pitch 为**角度**、range 米 | APIDOC + 运行时 |
| `getCaptureImg(opts)` → Promise dataURL | 无参、同步返回 `{image,width,height}`；需 `preserveDrawingBuffer=true`（否则打印提示并 undefined） | 运行时 |
| 相机事件 `on("CAMERA_CHANGED")` 式 | `changed/moveStart/moveEnd` 为 Event 对象 → `addEventListener` | 成员类型 |
| 飞图层前 `getLayerById` | 用 `layerManager.getLayer(id/对象)`（见 cimcube-layer）取层后再作 target | cimcube-layer 校准 |
