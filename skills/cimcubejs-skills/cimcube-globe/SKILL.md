---
name: cimcube-globe
description: "CIMCube 全局地球对象 Globe：地球显示/底色/地下颜色、光照、地形深度测试、水面流动、地形夸张、半透明，以及椭球读取。当用户需要控制地球显示/底色/地下颜色、光照、地形深度测试、水面流动、地形夸张、半透明，或读取椭球时激活。"
---

# CIMCube 全局地球对象 Globe

> 对象访问路径 **`map.globe`** —— 它是 `Glodon.CIMCube` 主类的**只读成员**（`globe : Glodon.CIMCube.Globe`），不是 `scene` 的成员，也不是 Cesium 原生 Globe。
>
> **SKILL.md 是纯路由索引**：本页只保留「结论先行成员表 + Quick Start + 路由表」；逐成员使用建议、常见模式、分工与排查全部下沉到 `references/`。

## 结论先行（API_DOC + 运行时双重确认）

`Glodon.CIMCube.Globe` 是封装后的地球渲染对象，**只暴露 10 个成员，没有任何方法**：

| 成员 | 类型 | API_DOC 说明 |
|---|---|---|
| `show` | `boolean` | 地球是否显示 |
| `baseColor` | `Color` | 地球基础颜色 |
| `enableLighting` | `boolean` | 光照 |
| `shadows` | `number`（Cesium `ShadowMode` 枚举：0/1/2/3） | 地球阴影（DISABLED/ENABLED/CAST_ONLY/RECEIVE_ONLY） |
| `depthTestAgainstTerrain` | `boolean` | 地形深度测试 |
| `ellipsoid`（只读） | `Cesium.Ellipsoid` | 椭球体 |
| `showWaterEffect` | `boolean` | 水面流动效果 |
| `terrainExaggeration` | `number` | 地形夸张 |
| `translucency` | `GlobeTranslucency` 对象（经 `.enabled` 开关） | 地球半透明 |
| `undergroundColor` | `Color` | 地球地下颜色 |

> 颜色/椭球等 Cesium 类型统一取 **`Glodon.CIMCube.Cesium.*`**（SDK 内嵌 Cesium）。⚠️ 不存在 `Glodon.CIMCube.Color`；写颜色要用 `Glodon.CIMCube.Cesium.Color.fromCssColorString("...")`。

## Quick Start

```js
const map = new Glodon.CIMCube({ id: "cimcubeContainer", gisResourceUrl: "./dist/" });
const G   = map.globe;   // Glodon.CIMCube.Globe

G.show                 = true;
G.enableLighting       = true;    // 光照
G.baseColor            = Glodon.CIMCube.Cesium.Color.fromCssColorString("#1b5e9a"); // 无影像地区/海洋底色
G.depthTestAgainstTerrain = true; // 地形深度测试
G.showWaterEffect      = true;    // 水面流动效果
G.terrainExaggeration  = 1.0;     // 地形夸张倍数
G.translucency         = false;   // 半透明
G.undergroundColor     = Glodon.CIMCube.Cesium.Color.fromCssColorString("rgba(120,180,255,0.25)");
```

裸初始化（不加任何图层）时地球表面没有影像，显示 `baseColor` 底色，属正常现象；影像/地形/模型图层一律通过 `map.layerManager` 添加（见 **cimcube-layer**）。

## 路由表（按需读取，勿整包加载）

| 你要查什么 | 读这个文件 |
|---|---|
| 10 个成员逐个的使用建议与注意事项 | `references/members.md` |
| 可运行模式：地表挡与不挡 / 夜景光照 / 季节底色 | `references/patterns.md` |
| 与 scene·layerManager·camera·Math 的分工边界、性能建议、排查表 | `references/pitfalls.md` |

## See Also

- **cimcube-setup** — `map.globe` 的来源与初始化、裸球为何没有影像
- **cimcube-scene** — 大气/天空盒/反射盒/雾/阴影/后期水等 scene 环境能力（不要把大气配置写到 Globe）
- **cimcube-realistic-rendering** — 真实感管线与光源（Bloom/体积云/大气散射/高度雾/体积光/自动曝光）
- **cimcube-layer** — ImageryLayer 影像底图、TerrainLayer 地形（LayerManager 提供 `getTerrainLayer(s)` 等）
- **cimcube-camera** — 相机定位/下钻/视角与 Globe 深度测试的配合
- **cimcube-global** — 经纬度与坐标互转（替代手动操作 `ellipsoid`）
- **cimcube-graphic** — 图元贴地/z-fighting（`depthTestAgainstTerrain` 与 Graphic 样式共同决定图元是否贴地/是否闪）

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
