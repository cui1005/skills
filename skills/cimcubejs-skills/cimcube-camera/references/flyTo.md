# flyTo — 平滑飞行

入口 `map.camera`。
**签名是单对象**：

```
camera.flyTo({
  target,                       // ★ 必填。见下方 target 分支表
  flyToPoint: false,            // 精确飞到一个点（而非按包围球入镜）
  orientation,                  // {heading,pitch,roll}
  duration,                     // 飞行时长（秒）。不传走内核默认（约 3s 平滑）
  distFactor: 1,                // 包围球模式的距离因子
  complete,                     // 飞行完成回调
  cancel,                       // 飞行取消回调
  easingFunction,               // ⚠️ 只接受**大写下划线名字符串白名单**，不接受 Cesium.EasingFunction 枚举成员！
                                //    ✅ 实测可用：LINEAR_NONE、CUBIC_IN/OUT/IN_OUT、QUARTIC_*、QUINTIC_*、SINUSOIDAL_*、EXPONENTIAL_*、CIRCULAR_*、ELASTIC_*、BACK_*、BOUNCE_*（28 个）
                                //    ❌ 被拒：LINEAR、QUADRATIC_IN/OUT/IN_OUT（运行时查表缺失）
                                //    非法值行为：控制台 log "unknown easingFunction!" 且 **throw undefined**（不是 Error！catch 里读 e.message 会二次 TypeError，须用 String(e) 兜底）
  maximumHeight,                // 最大飞行高度
  timeAheadInSecond,            // 时间提前量（秒）
  decreaseRequesting,           // 飞行中减少请求（仅 decreaseRequestingWhileFlying=true 时才生效）
})
```

**target 分支（运行时源码实证）**：

| target 传入 | 处理 |
|---|---|
| 图层（带 `isTiles3DLayer`/`isGltfLayer`/`isDivLayer`/`isI3STilesLayer`/`isS3MTilesLayer`/`isGSMMLayer` 之一） | `getBoundingSphere()`；拿不到包围球且图层自身有 `flyTo()` 时直接调 `layer.flyTo()` |
| `Cartesian3` / 带 `x` `y` `z` 的对象 | 原样当作内核坐标使用（**x/y/z 是 ECEF 米**，别拿它写经纬度） |
| `Rectangle` | ⚠️ **仅 `flyToPoint: true` 分支可靠**（飞到矩形中心上空）。默认包围球分支不解析 Rectangle，实测飞到**北极点（lon 0 / lat 90 / 地下）**。按矩形范围入镜改用 `flyToBoundingSphere({ bbox: [...] })` |
| 其他（含 `{lon,lat,height}` 普通对象） | 自动 `Cartesian3.fromDegrees(lon,lat,height)` |

**flyToPoint 语义差异**：

- `flyToPoint: true` → 等价内核 `camera.flyTo({ destination: 目标点, orientation, duration/complete/cancel/… })`：精确落点并套用你的姿态。**与点目标（经纬高/Cartesian3）搭配最可控**。
- 默认（不设/`false`）→ 构造以目标为中心的包围球（图层=其包围球半径×`distFactor`；点目标半径≈0.001）再调 `flyToBoundingSphere`，此时 `orientation` 作为**围绕目标的 HeadingPitchRange 偏移**（弧度）；不传 orientation 时默认从正上方俯视。

```js
// ① 精确飞到北京 3km 高度并保持 45° 斜视（推荐用法）
map.camera.flyTo({
  target: { lon: 116.4074, lat: 39.9042, height: 3000 },
  flyToPoint: true,
  orientation: { heading: 0, pitch: -Math.PI / 4, roll: 0 }, // 弧度
  duration: 3,
  complete: () => console.log("到达"),
});
// ② 鸟瞰某个已加载的图层（自动包围球，默认从上方看）
const layer = map.layerManager.getLayer("tiles3d-xxx"); // 任意 Tiles3D/Gltf/Div/I3S/S3M/GSMM 图层
map.camera.flyTo({ target: layer, duration: 2 });
// ③ 从斜上方看图层（orientation 被当作围绕目标的 HPR 偏移）
map.camera.flyTo({
  target: layer,
  orientation: { heading: Math.PI / 4, pitch: -Math.PI / 3, roll: 0 },
});
// ④ 矩形范围：Rectangle 只能走 flyToPoint 分支（默认包围球模式会飞到北极点）；
//    想按范围整体入镜用 flyToBoundingSphere({bbox})
map.camera.flyTo({
  target: Cesium.Rectangle.fromDegrees(112.78, 21.92, 112.90, 22.02),
  flyToPoint: true, duration: 2.5,
});
map.camera.flyToBoundingSphere({ bbox: [112.78, 21.92, 0, 112.90, 22.02, 0] });
```

> 📌 姿态统一传**弧度**（与构造 `center.heading/pitch/roll` 相同）；如传度数会被当弧度解释（-45 ≈ -58.3° 的老坑）。度→弧度可用 `Glodon.CIMCube.Global.Math` 工具或直接 `Math.PI/180 * deg`。
