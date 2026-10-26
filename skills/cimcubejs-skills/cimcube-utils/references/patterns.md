# Utils 公共初始化 · 组合模式

> 归属 `cimcube-utils`。本文件是跨功能族的**公共约定与组合套路**（写前必读）：公共初始化 + 组合模式。
> 逐类路由与**模块边界**见 `../SKILL.md`（①②③ 表 + See Also）；**坑点清单**（文档口径 / 废弃类 / 臆造 API）与**性能红线**见 `references/pitfalls.md`。
> 命名空间运行时 `Object.keys` 实测共 **28 类**。

## 一、公共初始化

```js
const U   = Glodon.CIMCube.Utils;
const DEF = Glodon.CIMCube.Global.Defines;   // MeasureMode / MapActionType / LengthUnit / AreaUnit ...

const map = new Glodon.CIMCube({
  id: "cimcubeContainer", gisResourceUrl: "./dist/",
  center: { lon: 116.4, lat: 39.9, height: 3000, pitch: -Math.PI / 4 },
});
```

**构造签名三类**（务必区分，别对静态类 `new`；逐类归属见 `../SKILL.md`）：

- 交互/场景型：`new X(map[, opts])`——绝大多数类（Measure、Snap、Clip 族、Cut/Excavation、分屏、HeatMap、ModelMarkup、各 Service、Cloner）。
- 纯静态类：`Utils.CoordinateTransform` / `Utils.SightLimitCalculate` / `Utils.BufferTool` / `Utils.MeasureUtils.*`——**直接调静态方法，不要 `new` 后找实例方法**。
- `Utils.POISearchService`：**只收 options（不含 map）**。

## 二、组合模式（真实 API）

### 模式 A：左侧工具栏（工具切换 = 销毁前一个）

```js
const ToolBar = {
  current: null,
  activate(mode, opts = {}) {
    this.current?.deactivate?.();
    this.current?.dispose?.();       // Measure 必须 dispose 才会清标注/释放
    this.current = new U.Measure(map, mode, opts);
    this.current.activate();
  },
  none() { this.current?.dispose?.(); this.current = null; },
};
document.getElementById("tool-m-dist").onclick = () => ToolBar.activate(DEF.MeasureMode.Ground_Distance);
document.getElementById("tool-m-area").onclick = () => ToolBar.activate(DEF.MeasureMode.Ground_Area);
document.getElementById("tool-m-azim").onclick = () => ToolBar.activate(DEF.MeasureMode.Azimuth);
```

### 模式 B：程序化"输入点 → 出结果"（不经 UI 手点）

```js
const m = new U.Measure(map, DEF.MeasureMode.Space_Distance, {});
m.on(DEF.MapActionType.FINISH, (e) => console.log("finished", e.mode));
m.setPosition({ lon: 116.40, lat: 39.90, height: 30 }); // 等价一次左键拾取
m.setPosition({ lon: 116.41, lat: 39.91, height: 80 });
m.completion();                 // 等价右键，结束当前量测 -> FINISH
m.dispose();
```

### 模式 C：盒剖 + 取盒（用户框选一个基坑范围查看地下）

```js
const box = new U.BoxClip(map);
const dragger = new U.BoxDragger(map);
dragger.start((r) => {
  box.clipEx(r.posAInWorld, r.posBInWorld, r.height, 0);   // 0 = 贴地表向下看地下
  dragger.dispose();
});
// 需要恢复：box.clear(); box.dispose();
```

### 模式 D：取点画轮廓 → 开挖（TakeCoordinates + ModelExcavation）

```js
const D = Glodon.CIMCube.Global.Defines;
const ex = new U.ModelExcavation(map, { depth: 20 });
const tc = new Glodon.CIMCube.Controls.TakeCoordinates(map.controls);
tc.on(D.MapActionType.FINISH, (e) => {
  ex.setPositions(e.coordinates);      // e.coordinates: [{lon,lat,height},...]
  ex.doAnalysis();
  tc.dispose();
});
tc.active(0, true);                     // 右键结束（收集 ≥1 点）
// 结束开挖：ex.clear(); ex.dispose();
```

### 模式 E：吸附开关 + 精准画线（Snap + TakeCoordinates）

```js
const snap = new U.Snap(map, { typeList: [D.SnapType.ENDPOINT, D.SnapType.MIDPOINT] });
snap.enable();                          // 之后所有取点都贴到模型端点/中点
const tc = map.controls.takeCoordinates;
tc.on(D.MapActionType.ADD, (e) => drawPointMarker(e.position));
tc.active(2, true);
// 完成：snap.disable();  tc.clear();
```

### 模式 F：道路红线退让缓冲（TakeCoordinates + BufferTool + Graphic）

```js
const G = Glodon.CIMCube.Graphic;
const { Cartographic } = Glodon.CIMCube.Global.Math;
const ring = await pickPolylineAsync(map);                       // [{lon,lat,height}]
const cartoPoints = ring.map(p => Cartographic.fromDegrees(p.lon, p.lat));
const buffer = U.BufferTool.generatePolylineBuffer(cartoPoints, 30);   // Array<Cartographic>，30 米
const bufferLayer = map.layerManager.addLayer({
  type: D.LayerType.GRAPHIC, id: "buffer", name: "道路退让缓冲",
});
bufferLayer.addGraphic(new G.PolygonGraphic({                    // 显示缓冲面
  // ★ 图元 positions 是 [lon,lat,height] 数组；Cartographic 是弧度，需转回度
  positions: buffer.map(c => [c.longitude * 180 / Math.PI, c.latitude * 180 / Math.PI, 0]),
  material: "rgba(0,180,255,0.35)",                              // ★ 面样式键是 material，没有 polygon:{color}
}));
```

### 模式 G：对比工具做 A/B 切换（正确启停顺序）

```js
let cur = null;
function compareA() {
  cur?.stop?.(); cur?.dispose?.();
  cur = new U.MapSplit(map, map.layerManager, { mode: "horizontal", viewPortLayersVisible: lsA });
  cur.begin();
}
function compareB() {
  cur?.stop?.(); cur?.dispose?.();
  cur = new U.MultiViewPort(map, map.layerManager, { mode: "vertical", viewPortLayersVisible: lsB });
  cur.begin();
}
```

### 模式 H：自定义状态条（Service 数据 → 自己的 DOM）

```js
const ss = new U.StatusBarService(map);
ss.addEventListener({ widget: bar, method: (lng, lat, h, hd, p, fps, ms) => render(bar, lng, lat, h, hd, p) });
ss.start();
```

