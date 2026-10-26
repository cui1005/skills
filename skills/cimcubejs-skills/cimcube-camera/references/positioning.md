# 定位与姿态 — setView · flyToBoundingSphere · lookAt

入口 `map.camera`。

> `flyTo`（最常用的平滑飞行，单对象参数 + 单位坑集中）单独见 `references/flyTo.md`。

## `setView(options)` — 无动画瞬间定位

```
camera.setView({
  destination,   // {lon,lat,height} | Rectangle({west,south,east,north}) | Cartesian3
  orientation    // {heading,pitch,roll} 弧度
})
```

运行时转换：destination 已是 `Cartesian3`/`Rectangle` → 原样用；否则含 `lon/lat/height` → `fromDegrees`；否则当 `{west,south,east,north}` → `Rectangle.fromDegrees`。orientation 直接透传内核 `camera.setView`（弧度）。

```js
// 无动画直接站到北京上空 45° 俯视
map.camera.setView({
  destination: { lon: 116.4074, lat: 39.9042, height: 3000 }, // 度/度/米
  orientation: { heading: 0, pitch: -Math.PI / 4, roll: 0 },  // 弧度！
});
```

## `flyToBoundingSphere(options)` — 按包围球飞行定位

**单 options 参数**（"双参 + 图层直接传"不成立）：

```
camera.flyToBoundingSphere({
  center,             // 包围球球心：[lon, lat, height]（度/度/米）或 Cartesian3
  radius,             // 包围球半径（米）。与 center 成对出现
  bbox,               // 包围盒：两个对角点经纬高平铺 [lon,lat,h, lon,lat,h]（6 个数）
  duration: 3,        // 飞行时间（秒）
  offset,             // {heading, pitch, range} — heading/pitch 单位【度】(内部转弧度)！range 单位米
})
```

- `center+radius` 与 `bbox` 二选一，缺一不可（都不传直接 return）。
- `offset` 的默认姿态=正俯视（heading 0、pitch -90°）；**传了 offset 但漏写 range 时 range 按 0 处理**，因此要么带 `range`（目标到相机的距离，米），要么省略整个 offset。
- `duration` 默认 **3**。

```js
// 飞到一片区域正上方，半径 2000m
map.camera.flyToBoundingSphere({
  center: [116.39, 39.90, 0],
  radius: 2000,
  duration: 3,
  offset: { heading: 0, pitch: -90, range: 4000 }, // pitch 是角度(度)，range 是米
});
// 用两角包围盒
map.camera.flyToBoundingSphere({
  bbox: [116.35, 39.85, 0, 116.45, 39.95, 0],
  offset: { heading: 30, pitch: -60, range: 5000 },
});
```

> ⚠️ 本方法 `offset.heading/pitch` 是**度**，而 `flyTo` 的 `orientation` 是**弧度**——同一组字段名两套单位，最易踩坑。

## `lookAt(target, offset)` — 面向某目标（相机不动）

```
camera.lookAt(target, offset)
  target  // Cartesian3 | {lon,lat,height}
  offset  // Cartesian3 | HeadingPitchRange（ENU 偏移）；也可写 {lon,lat,height}（运行时 fromDegrees 后按 Cartesian3 语义）
```

适合"相机不动、看着某物"的姿态控制（跟踪场景常用）。目标/偏移优先使用 SDK 其他接口返回的 `Cartesian3`（如 `pickEllipsoid` 结果）；需要从业务视角观察物体时优先用 `flyTo`/`flyToBoundingSphere` 更直观。
### lookAt 的副作用与解除（务必成对使用）

- **必须显式解除，否则鼠标/控制一直被锁**：
  - 解除调用 `camera.lookAtTransform(Matrix4.IDENTITY)` 把参考系清零。**`map.camera`（CIMCube 包装层）未暴露 `lookAtTransform`**；底层真 Cesium 相机在 `map.camera._camera`，故释放应写 `map.camera._camera.lookAtTransform(Glodon.CIMCube.Cesium.Matrix4.IDENTITY)`。
  - ⚠️ `setView` / `flyTo` **不会**自动清零参考系：若只调用它们而不先 `lookAtTransform(IDENTITY)`，相机仍相对锁定系运动，鼠标受限依旧。正确顺序：先 `lookAtTransform(IDENTITY)` 再 `setView` 复位姿态。
  - `trackedModel` 赋值时内部也走 `lookAt` 其包围球中心；解除需同样 `lookAtTransform(IDENTITY)` 或把 `trackedModel = undefined`。
