# 避坑记录与性能建议（真实感渲染 / 光源）

> 本文件对应 SKILL.md 路由表的「易错写法对照、性能取舍」行。

## 避坑记录（易错写法 → 真实 API）

1. **`map.scene.RealisticRendering`（大 R）与 `map.scene.RealisticRendering.Bloom` 不存在** → `map.scene.realisticRendering` + 小写 camelCase 子模块；`Glodon.CIMCube.Scene` 静态只有 `ReflectBox/PointLight/SpotLight/DirectionalLight/GroundSkyBox` 五个类可 new，**没有** `Bloom/SkyAtmosphere/SkyLight/VolumetricClouds/RealisticRendering/EyeAdaptation/LightShaft/ExponentialHeightFog` 构造函数。
2. **SkyLight 无 `topColor/bottomColor`** → 只有 `color`（单色）+ `intensity`。
3. **Bloom 无 `threshold/radius`** → `RR.bloom` 只有 `enabled/intensity`。Bloom 的“范围感”交给 `RR.eyeAdaptation` 曝光与 HDR。普通（非真实感）泛光用 `postProcessStages` 的 Cesium bloom（见 `cimcube-scene`）。
4. `RR.origin.lon / .lat / .height` 直接赋值，**无 `set()`**。
5. 构造 `new DirectionalLight(opts)`，用 `position+targetPosition` 确定方向。
6. `SpotLight` 的 `angle`（单开合角，度）+ `distance` + `targetPosition`。缺 `targetPosition` 构造会失败（返回空对象并告警）。
7. **光源构造即发光？不对**：三个光源类 `new` 后只建立对象，**必须 `map.scene.addLightSource(light)`**，`removeLightSource(light)` 移除。
8. 光源「物理单位」只在 `enableRealisticRendering` 开启后生效；关闭时强度为普通缩放值（默认 1）。
9. 初始预置**不在构造器**（`new CIMCube({ globalSettings })` 会被静默忽略），也不在 `sceneSetting`——通过 `Glodon.CIMCube.createScene(map, { globalSettings: { realisticRendering: {...} } })` 应用。
10. **开启真实感渲染 ≠ 必须配光源**：`map.scene.enableRealisticRendering = true` + 子模块即生效，光照默认由 `RR.skyLight`（天光）与 `RR.skyAtmosphere` 提供。`DirectionalLight / PointLight / SpotLight` 只是**按需叠加**的额外打光，不要当成开启真实感的前置条件一并加上。唯一例外：`RR.lightShaft`（体积光束）的方向由 `DirectionalLight` 提供，要用它才必须有平行光。
11. **★ 前置硬性要求：`enableRealisticRendering` 前必须先开 `logarithmicDepthBuffer` 与 `depthTestAgainstTerrain`**——二者缺一，真实感管线下的深度精度与地形遮挡会错乱：

    ```js
    map.scene.logarithmicDepthBuffer = true;      // 对数深度缓冲，真实感管线下必需
    map.globe.depthTestAgainstTerrain = true;     // 地形深度测试，属 map.globe 而非 map.scene
    map.scene.enableRealisticRendering = true;        // 之后才开总开关
    ```

    注意 `depthTestAgainstTerrain` 挂在 **`map.globe`** 上（见 `cimcube-globe`），不要写成 `map.scene.depthTestAgainstTerrain`。

12. **★ 真实感与天空盒（skyBox / firmamentSkyBox）互斥，不要同时使用**：`enableRealisticRendering = true` 后，天空由 `RR.skyAtmosphere`（大气散射）与 `RR.skyLight`（天空光）接管，`map.scene.skyBox` / `map.scene.firmamentSkyBox` 会被覆盖而**看不见**。若需要自定义背景天空，应**关闭真实感**用 `map.scene.skyBox`；若需要真实感画质，则不要配天空盒，改用 `RR.skyAtmosphere` 等子模块调天空效果。

13. **★ `RR.origin` 必须落在场景的实际地理位置，否则整片被灰雾笼罩、什么都看不见**：真实感渲染只覆盖以 `RR.origin`为圆心的**有限范围（非全球）**，大气散射 / 高度雾 / 光照效果都以此为原点。初始化时把 `RR.origin` 设到待展示区域的中心。

## 性能建议

| 场景 | 推荐 |
|---|---|
| 大范围地形/城市 | `RR.volumetricClouds` 关或 `cloudQuality=0`，改用 `map.scene.nephogram` 云图（见 `cimcube-scene`） |
| 室内/小场景 | 点光/聚光 ≤ 3 盏；`RR.bloom` / `RR.lightShaft` 择一 |
| 真实感模式发黑/发灰、什么都看不见 | **先查 `RR.origin` 是否对齐场景实际位置（见坑 13，偏差几度即整片灰雾）**；再查 `RR.eyeAdaptation`、`RR.skyLight.intensity`、`DirectionalLight.intensity`（lux 偏大先给 1–3） |
| 体积云 | `cloudQuality` 0/1/2 逐级变贵；`cloudCoverage` 越大越贵 |
