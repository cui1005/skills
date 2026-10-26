# 避坑记录与性能建议（场景渲染）

> 本文件对应 SKILL.md 路由表的「易错写法对照、性能取舍」行。改 `map.scene` 环境项前建议先扫一遍。

## 避坑记录（易错写法 → 真实 API）

1. `map.scene.skyBox.mode =Glodon.CIMCube.Global.Defines.SkyBoxMode.DUSK`、`map.scene.skyBox.show = true`。mode 支持 **17 个 SkyBoxMode 预设**。
2. `new ReflectBox({mode/show/sources/useHDRCubeMap})`；读换 `map.scene.reflectBox` 实例或赋新对象。
3. 初始渲染预置不在 `sceneSetting`，也**不在构造器**——`globalSettings` / `mapLayers` / `defaultView` 三个字段由 `Glodon.CIMCube.createScene(map, { globalSettings, mapLayers, defaultView })` 统一应用。
4. **真实感渲染总开关 / 子模块 / 光源的坑**（`map.scene.RealisticRendering` 大 R 不存在、`RR.bloom` 无 threshold、光源必须 `addLightSource` 等）见 `cimcube-realistic-rendering` 避坑节。
5. **场景物理的坑**（`map.scene.ScenePhysics` 属性直改不存在、程序化刚体需 `bindVisual` 等）见 `cimcube-physics` 避坑节。

## 性能建议

| 场景 | 推荐 |
|---|---|
| 大范围地形/城市 | `atmosphere` 常规 + `nephogram` 云图代替真实感云；`shadows` 只对近景开、`shadowRange` 收窄 |
| 玻璃/金属 | `map.scene.reflectBox` 用 HDR 预设（`HDR_*`），代价高于 JPG；不追求反射可设 `environmentReflection:'SKY'` |
| 高帧率 | 关 `softShadows`、`hardwareAA` 改 MSAA 4x（勿与后处理并存）、`debugShow*` 全关、`targetFrameRate` 合理值 |
| 阴影闪烁 | `castShadowAlphaThreshold` + `shadowUpdateFrequency` 调低 + `shadowOptimizeWhenIdling` 开 |
| 真实感/物理 | 见 `cimcube-realistic-rendering` 与 `cimcube-physics` 性能节 |
