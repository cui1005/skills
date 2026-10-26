# Utils 裁剪开挖族 — ModelCut / ModelExcavation

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

## 相关枚举

```js
const D = Glodon.CIMCube.Global.Defines;
D.ModelCutMode;       // {INNER:0, OUTER:1}           裁剪内部 / 裁剪外部（ModelCut 用）
```

## 一、ModelCut — 模型区域裁剪显示（按多边形 INNER/OUTER）

不是"沿平面切开保留几何"，而是**用屏幕/地理多边形把目标图层裁剪成内部或外部可见**（`D.ModelCutMode.INNER=0` 只显示多边形内，`OUTER=1` 只显示多边形外）。

| 项 | 内容 |
|---|---|
| 构造 | `new U.ModelCut(map, { cutMode?: D.ModelCutMode, region?: Array, layers?: Array<(Tiles3DLayer|GltfLayer)>, idPrefix?, edgeColor?, edgeWidth? })` |
| `region` | 裁剪面顶点数组，顶点格式 `[lon, lat, height]`（度、米）；单个区域是二维数组，**多区域用嵌套数组**（例见下） |
| `addRegion(value, id?, updateNow?)` / `removeRegion(id, updateNow?)` | 增删区域 |
| 生命周期 | `activate()` 开启裁剪 / `deactivate()` 停止 / `dispose()` |
| 属性 | `cutMode` / `edgeColor` / `edgeWidth` / `layers` / `region` / `isActive`（只读） |

```js
const cut = new U.ModelCut(map, {
  cutMode: D.ModelCutMode.OUTER,            // 挖掉多边形内：显示外部
  layers: [s3mLayer],
  region: [[116.3, 39.9, 0], [116.36, 39.9, 0], [116.36, 39.95, 0], [116.3, 39.95, 0]],
});
cut.activate();
// cut.addRegion([[116.31, 39.91, 0], ...], 'region2', true);   // 追加区域
// 结束：cut.deactivate();  cut.dispose();
```

> ⚠️ 臆造（不可用）：`new U.ModelCut(map, { target, plane:{point, normal} })` / "沿平面切成两半" 不存在。真实语义是区域裁剪 `{ cutMode, region([lon,lat,height]多边形), layers }` + `activate/addRegion/removeRegion`；**无 plane 语义**。

## 二、ModelExcavation — 模型开挖分析

| 项 | 内容 |
|---|---|
| 构造 | `new U.ModelExcavation(map, { positions?, depth?, bottomTextureUrl?, facadeTextureUrl?, layers? })` |
| `positions` | 开挖范围顶点 `Array<GeoPoint>`：元素 `{lon, lat, height}`，**需逆时针**（语义即"多边形+深度挖洞"，见 APIDOC options.positions 注释） |
| `depth` | 开挖深度（米）；`bottomTextureUrl`/`facadeTextureUrl`：坑底/侧壁贴图 |
| 方法 | `doAnalysis()` 执行开挖；`clear()` 清除；`dispose()` |
| 便捷 setter | `setPositions(arr)` / `setDepth(v)` / `setBottomTexture(url)` / `setFacadeTexture(url)` |
| 属性 | `positions` / `depth` / `bottomTextureUrl` / `facadeTextureUrl` / `layers` / `isActive` / `region`（只读，执行后的实际区域） |

```js
const ex = new U.ModelExcavation(map, {
  positions: [
    { lon: 116.35, lat: 39.9, height: 0 },
    { lon: 116.36, lat: 39.9, height: 0 },
    { lon: 116.36, lat: 39.91, height: 0 },
    { lon: 116.35, lat: 39.91, height: 0 },   // 逆时针
  ],
  depth: 15,
  bottomTextureUrl: './tex/bottom.jpg',
  facadeTextureUrl: './tex/facade.jpg',
});
ex.doAnalysis();
// 结束：ex.clear(); ex.dispose();
```

> ⚠️ 臆造（不可用）：`new U.ModelExcavation(map, { polygon })` 不存在。真实为 `positions`（逆时针 `{lon,lat,height}`）+ `depth` + 贴图 URL，`doAnalysis()` 触发。
