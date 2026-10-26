# Globe 分工边界 · 性能建议 · 排查表

> 本文件对应 SKILL.md 路由表的「与 scene·layerManager·camera·Math 的分工边界、性能建议、排查表」行。

> 成员语义见 `members.md`，模式见 `patterns.md`。

## 与相邻模块的分工（避免再次混淆）

| 你想要的 | 正确对象 |
|---|---|
| 地球外壳开关/底色/光照/深度测试/水面流动/地形夸张/半透明/椭球 | `map.globe`（本技能） |
| 天空、大气、云、Bloom、真实感光照 | `map.scene`（`map.scene.realisticRendering.*`） |
| 影像底图、地形数据、3D 瓦片/模型 | `map.layerManager`（ImageryLayer / TerrainLayer 等） |
| 相机飞入飞出、地下钻入限制 | `map.camera` |
| 经纬度 ↔ 世界坐标换算 | `Glodon.CIMCube.Global.Math` |

## 性能建议

- `showWaterEffect = true` 会持续做水面流动渲染，静态场景记得关。
- `terrainExaggeration` 非 1 时地形起伏被放大，飞行/落点高度判断要同步考虑放大倍数。
- 大范围高频取高/拾取不是 Globe 的能力，不要在 `map.globe` 上找这类 API。

## 常见问题排查表

| 现象 | 原因/处理 |
|---|---|
| 想"贴地/贴地形正确"或"图元被山挡住" | 检查 `G.depthTestAgainstTerrain` |
| 地球底色/无影像区域一片黑 | 无影像图层时显示 `baseColor`，用 `G.baseColor = ...Color.fromCssColorString(...)` 调整（裸球初始不显示影像属正常） |
| 想精确取某点地形高 | **Globe 没有取高 API**（10 个成员全是属性、无方法）：地形数据图层见 **cimcube-layer**，坐标/高程换算见 **cimcube-global** |
