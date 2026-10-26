# Animation 域公共初始化与常见组合模式

> 公共初始化全族通用；组合模式按业务分组（与 SKILL.md 节号 ①②③ 对应）。

## 公共初始化（所有 references 示例的前提）

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer", gisResourceUrl: "./dist/",
  center: { lon: 116.4, lat: 39.9, height: 3000, pitch: -Math.PI / 4 }, // pitch 弧度
});
const E = Glodon.CIMCube.Animation;                 // 动画命名空间
const Cesium = Glodon.CIMCube.Cesium;               // SDK 内嵌 Cesium 内核（颜色/几何枚举等）
// 命名空间前缀：
//   E.Effect.* / E.LineEffect.* / E.PlaneEffect.* / E.ModelEffect.*
//   E.ParticleEffect.* / E.WeatherEffect.* / E.PointEffect.*
```

## ① 漫游与场景特效实体组合模式

### 模式 A：领导参观一键导览（相机漫游）

```js
function CEOVisit() {
  applyPresetScene("dusk");
  const tour = new E.Effect.CameraRoaming(map.camera, { timeAheadInSecond: 3 });
  tour.setPath(CEO_WAYPOINTS);
  tour.setDuration(60);
  tour.setLoopMode(false);
  tour.play();
}
```

### 模式 B：领导实地巡查（模型沿轨迹漫游 + 相机跟随）

```js
function SitePatrol() {
  const track = new E.Effect.TrackRoaming(map, {
    pathList: [
      { url: "./models/worker.glb", speed: 3, data: WALK_PATH },
    ],
    eyeHeight: 1.8, cameraRange: 5, showPath: true,
  });
  track.play();
}
```

## ② 视觉图形组合模式

### 模式 A：领导参观一键导览（飞线 + 雷达 + 相机漫游）

```js
function CEOVisit() {
  applyPresetScene("dusk");
  const beltLines = new E.LineEffect.FlyLines(map, {
    positionList: BRI_ROUTES,
    lineColor: Cesium.Color.RED,
    lineWidth: 2,
    speed: 12,
    percent: 0.15,
  });
  beltLines.show();
  const radar = new E.PlaneEffect.Radar(map, {
    originPosition: URUMQI_PORT,
    color: Cesium.Color.fromCssColor("#f57c00"),
    radius: 800,
    duration: 4,
  });
  radar.show();
  // 相机漫游见 cimcube-animation references/Effect-roaming.md
  registerAnimations(beltLines, radar);
}
```

### 模式 B：BIM 爆炸 + 模型单体高亮组合展示

```js
function BIMShowcase(building) {
  const exp = new E.ModelEffect.ModelExplode(map, {
    tiles3DLayer: building,
    duration: 3,
    offset: { x: 0, y: 8, height: 2 },
  });
  exp.doExplode();
  // 爆炸完毕后，对某些 feature 做高亮
  setTimeout(() => {
    const mi = new E.ModelEffect.ModelInteroperation(map, {
      tiles3DLayer: building,
      interoperationList: [
        { ids: ["feature-window-101"], offset: { x: 0, y: 0, height: 0 } },
      ],
      duration: 1,
    });
    mi.play();
  }, 3000);
}
```

### 模式 C：大量 POI + 飞线大屏（物流链路）

```js
function logisticsDashboard() {
  const fl = new E.LineEffect.FlyLines(map, {
    positionList: LOGISTICS_PAIRS,
    lineColor: Cesium.Color.RED,
    lineWidth: 2,
    endHeight: 300,
    speed: 15,
  });
  fl.show();
  // 散点扩散 → PointEffect.Scatter（references/PointEffect.md）
  const sc = new E.PointEffect.Scatter(map, {
    positions: HUB_POINTS,
    color: Cesium.Color.YELLOW,
    size: 20,
    spreadRatio: 4,
    duration: 2,
    totalWaves: 3,
  });
  sc.show();
}
```

## ③ 粒子·天气·点动态组合模式

### 模式 A：大屏一键场景预设（白天/暴雨/下雪/起雾）

```js
const Presets = {
  "day":  () => { stopAllWeather(); /* 关闭天气类 */ },
  "rain": () => {
    stopAllWeather();
    const r = new E.WeatherEffect.Rain(map, { darkness: 0.6, density: "great" });
    r.show(); registerWeather(r);
  },
  "snow": () => {
    stopAllWeather();
    const s = new E.WeatherEffect.Snow(map, { darkness: 0.6, density: "normal", thickness: 0.3 });
    s.show(); registerWeather(s);
  },
  "fog":  () => {
    stopAllWeather();
    const f = new E.WeatherEffect.Fog(map, { darkness: 0.6, lightAttenuation: 2, visualDistance: 800 });
    f.show(); registerWeather(f);
  },
};
document.getElementById("preset").onchange = (e) => Presets[e.target.value]();
```

### 模式 B：火情演练 — 发现火情 → 闪烁 + 火焰 + 烟雾

```js
async function fireDrill(firePos) {
  const tw = new E.PointEffect.Twinkle(map, {
    positionList: [{ x: firePos.lon, y: firePos.lat, z: 2 }],
    size: 20, image: "./assets/warn.png", breathDuration: 1,
  });
  tw.show();
  await delay(400);
  const fireP = new E.ParticleEffect.Fire(map, {
    position: firePos, fireRadius: 60, windStrength: 0.3,
  });
  const smoke = new E.ParticleEffect.Smoke(map, {
    position: { ...firePos, height: firePos.height + 6 },
    smokeRadius: 20, density: 0.5, thickness: 3,
  });
  fireP.play(); smoke.play();
}
```

### 模式 C：大量 POI 实例化（>1000 个）

```js
// 不要用 GraphicLayer + N 个 BillboardGraphic，用 InstanceBillboard
const poiList = await fetchPOIs();  // 10000 个 POI
const ib = new E.PointEffect.InstanceBillboard(map, {
  positionList: poiList.map(p => ({ lon: p.lon, lat: p.lat, height: 10 })),
  image: "./assets/poi.png",
});
ib.show();
```
