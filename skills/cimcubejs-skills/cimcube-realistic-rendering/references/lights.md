# 光源（DirectionalLight / PointLight / SpotLight）

> 本文件对应 SKILL.md 路由表的「三类光源的 options / 成员 / 注册与移除」行。

构造签名统一为 **`new Glodon.CIMCube.Scene.XXX(options)`**（options 直接传对象，**没有 scene 参数**）。构造后必须 **`map.scene.addLightSource(light)`** 注册才参与光照（`type` 分别为 `'directional'/'point'/'spot'`，每类上限 16，超限警告）。

```js
const spot = new Glodon.CIMCube.Scene.SpotLight({
  position: { lon: 116.397, lat: 39.909, height: 12 },      // 光源所在经纬高(米)
  targetPosition: { lon: 116.397, lat: 39.908, height: 0 }, // 照射点（决定朝向）
  color: '#ffffff', intensity: 1,                           // 物理单位 cd（真实感模式）
  distance: 10,     // 光照距离(米)
  angle: 45,        // 开合角(度，单值)
  attenuation: 0.2, // 距离衰减
});
map.scene.addLightSource(spot);
spot.color = '#ffcf9f';   // 运行时可改
map.scene.removeLightSource(spot);
```

| 类 | options（默认值） | 成员要点 |
|---|---|---|
| `DirectionalLight` | `position`（{lon,lat,height}，必需）、`color:'#ffffff'`、`intensity:1`；可选 `targetPosition`（否则朝场景原点） | 平行光（太阳）。`position` 只决定方向。真实感模式下 `intensity` 为照度(lux) |
| `PointLight` | `position`（必需）、`color:'#ffffff'`、`radius:10`（米，作用半径）、`attenuation:0.5`、`intensity:1` | 点光源，全向衰减。真实感模式 `intensity` 为发光强度(cd) |
| `SpotLight` | `position` + `targetPosition`（均必需，缺 targetPosition 直接返回空对象）、`color:'#ffffff'`、`distance:10`、`angle:45`（度）、`attenuation:0.2`、`intensity:1` | 聚光灯 |

共有实例成员：`position / targetPosition / color / intensity`（可写），`positionWorld / targetPositionWorld / directionWorld / cesiumColor`（内部算好只读）、`updateEvent`（Cesium.Event，位置变化时触发场景刷新）。改位置/颜色后自动刷新。

> 打光建议：真实感开启后光照才按物理单位表现（普通模式只是平行强度缩放）；亮度过低/过曝先查 `enableRealisticRendering` 与 `RR.eyeAdaptation`。光源要投影需 `map.scene.shadows = true`（阴影细节见 `cimcube-scene`）。
