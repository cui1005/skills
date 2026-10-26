# 底图与地形 — ImageryLayer · TerrainLayer

入口 `map.layerManager`。

## ImageryLayer 影像底图

```js
const img = new Glodon.CIMCube.Layer.ImageryLayer({
  id: "img", name: "影像", url: "http://.../{z}/{x}/{y}.png",
  imageryType:Glodon.CIMCube.Global.Defines.ImageryLayerType.XYZ,   // ★ imageryType，不是 type
  zIndex: 0,
  options: {
    rectangle: { west: 100, south: 20, east: 120, north: 40 }, // 范围
    minimumLevel: 1, maximumLevel: 18,
    brightness: 1, contrast: 1, hue: 0, saturation: 1, gamma: 1,
    accessToken: { field: "tk", value: "xxxx" },   // 仅内置天地图
    zoomOffset: 0,                                  // WMTS 层级偏移
    enableColoring: false, blendColor: "#1E90FF", blendIntensity: 1,
  },
});
map.layerManager.addLayer(img);
```

- 成员：`show` `alpha` `zIndex` `brightness` `contrast` `hue` `saturation` `gamma` `enableColoring` `blendColor` `blendIntensity`
- 方法：`dispose()` `flyTo()` `getImageryProvider()`
- 常用 `imageryType`：`'xyz'` `'wmts'` `'wms'` `'tms'` `'tdt_img'`（天地图影像）`'tdt_vec'`（天地图矢量）`'gaode_img'` `'baidu_img'` `'arcgis_img'` `'tencent_img'` `'offline_img'`

## 内置底图（天地图 / 高德 / 百度 / ArcGIS / 腾讯 …）

`ImageryLayerType` 中的 `TDT_*`、`GAODE_*`、`BAIDU_*`、`ARCGIS_*`、`TENCENT_*`、`GOOGLE_*`、`SUPERMAP_*`、`OFFLINE_*` 均为 **SDK 内置底图**：已内置 provider 与默认 `tk`，**只需指定 `imageryType`，不用填 `url`**。枚举定义见 cimcube-global 的 `Defines.ImageryLayerType`（`cimcube-global/references/defines-enums.md`）。

```js
// ★ 添加天地图影像：直接套枚举，无需 url / tk
const tdtImg = new Glodon.CIMCube.Layer.ImageryLayer({
  id: "tdt-img", name: "天地图影像",
  imageryType:Glodon.CIMCube.Global.Defines.ImageryLayerType.TDT_IMG,   // 等价于字符串 "tdt_img"
});
map.layerManager.addLayer(tdtImg);

// ★ 添加高德矢量：同理，仅换枚举值
const gaodeVec = new Glodon.CIMCube.Layer.ImageryLayer({
  id: "gaode-vec", name: "高德矢量",
  imageryType:Glodon.CIMCube.Global.Defines.ImageryLayerType.GAODE_VEC, // "gaode_vec"
});
map.layerManager.addLayer(gaodeVec);
```

- **推荐用枚举常量**：`Defines.ImageryLayerType.TDT_IMG` 等价于字符串 `"tdt_img"`，用常量可避免拼写漂移（见 cimcube-global 枚举表）。
- **默认 tk**：内置天地图等走 SDK 自带 token；需用自有 token 时再在 `options.accessToken` 覆盖（`{ field: "tk", value: "xxxx" }`，仅内置天地图类源生效）。
- **自托管 / 第三方瓦片**才用 `XYZ` / `WMTS` / `WMS` / `TMS` 并在构造里填 `url`。
- 常见内置源枚举：`TDT_VEC/TDT_IMG/TDT_TER/TDT_CVA/TDT_CIA/TDT_CTA`、`GAODE_IMG/GAODE_VEC/GAODE_CIA`、`BAIDU_IMG/BAIDU_VEC`、`ARCGIS_IMAGE/ARCGIS_IMG/ARCGIS_GRAY`、`TENCENT_IMG`、`GOOGLE_IMG`、`SUPERMAP_IMAGE`、`OFFLINE_IMG/OFFLINE_MIX`（全量见 cimcube-global `Defines.ImageryLayerType`）。

## TerrainLayer 地形

```js
const terrain = new Glodon.CIMCube.Layer.TerrainLayer({
  id: "terrain", name: "地形",
  terrainType:Glodon.CIMCube.Global.Defines.TerrainLayerType.DEFAULT,
  url: "http://.../terrain",
  requestVertexNormals: false, requestWaterMask: false,
  requestMetadata: false, useAvailabilityLevels: false,   // 多地形场合建议 false
});
map.layerManager.addLayer(terrain);
```

- 成员：`show`；方法：`dispose()` `flyTo()`
- GeoServer 地形专用键：`service` `layers` `version` `tileSize` `format` `crs` `gridSet` `highest` `lowest` `maxLevel`；超图地形：`customRequestHeaders` `rectifyBound`
