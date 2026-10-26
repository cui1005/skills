# Globe 10 个成员的使用建议

> 本文件对应 SKILL.md 路由表的「10 个成员逐个的使用建议与注意事项」行。

> 成员清单与类型见 `SKILL.md`「结论先行」表；可运行模式见 `patterns.md`；分工与排查见 `pitfalls.md`。
> 访问路径统一为 `map.globe`（下称 `G`）。

### show（boolean）

整个地球显示/隐藏。做纯空天 HUD 或 2D 业务时可 `G.show = false`。

### baseColor（Color）

无影像覆盖区域的底色（含海洋）。用 `Glodon.CIMCube.Cesium.Color.fromCssColorString()` 赋值。

### enableLighting（boolean）

地球表面光照开关。开（`true`）时随太阳方向产生明暗；关闭则地表受光均匀。

### shadows（number，Cesium ShadowMode 枚举）

地球阴影模式，**不是配置对象**，而是数值枚举（对应 Cesium `ShadowMode`：`0` DISABLED / `1` ENABLED / `2` CAST_ONLY / `3` RECEIVE_ONLY）。当前默认值为 `3`（RECEIVE_ONLY）。赋值示例：`map.globe.shadows = 1;` // 开启 ENABLE。

### depthTestAgainstTerrain（boolean）—— 最常用

控制地表是否参与深度测试：

- **`true`**：地形能挡住后面的东西（如山的背面的建筑/图元不可见，图元贴地不"穿帮"）；
- **`false`**：透视地形（需要看到被地形挡住的标注/地下内容时用）。

出现"山背后还能看到图元"时先检查它是否被误设成了 `false`；出现"点取地下管点被球壳挡住"时再考虑关闭它并配合 `translucency`/`undergroundColor` 观察地下。

### ellipsoid（只读，Cesium.Ellipsoid）

椭球体参数对象（通常 WGS84），是 **Cesium 原生对象**，可按 Cesium API 读取。业务上经纬度↔世界坐标的换算建议直接用 **cimcube-global** 的 `Glodon.CIMCube.Global.Math` 工具，不必自己摆弄椭球矩阵。

### showWaterEffect（boolean）

水面流动效果开关。项目需要"水面动态流动"时开启；静态/内陆场景可关闭省资源。

### terrainExaggeration（number）

地形夸张倍数。数值越大起伏越明显（如 3 倍凸显地形高差）；不需要夸张保持 `1.0`。

### translucency（GlobeTranslucency 对象，经 `.enabled` 开关）

地球半透明。**不是 boolean**，而是一个 `GlobeTranslucency` 对象。开关半透明要写 `map.globe.translucency.enabled = true`（而非 `translucency = true`）；该对象还提供 `frontFaceAlpha` / `backFaceAlpha` / `*ByDistance` / `rectangle` 等字段控制正/背面透明度。做"看到地下模型/管线"场景时，将 `.enabled = true` 并与 `undergroundColor` 搭配使用。

### undergroundColor（Color）

地球地下部分的颜色（在 `translucency` 等配合下呈现）。
