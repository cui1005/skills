# Controls 上手：对象关系 + Quick Start

> 本文件对应 SKILL.md 路由表的「第一次上手 / 要可运行骨架」行。

> 相关：`references/take-coordinates-api.md`（方法与属性全表）、`references/events.md`（事件）、`references/snap.md`（吸附）、`references/patterns.md`（业务封装）

## 一、对象关系总览（先建立正确心智模型）

真实 SDK 只有一套取点控件，没有任何业务封装：

```
map（CIMCube 实例）
└── map.controls（拾取控制器，CIMCube 构造时自动创建，类未在 APIDOC 单独成页）
    ├── 内部注册了全部鼠标事件（左/右/中键、双击、移动），每帧做场景拾取
    ├── takeCoordinates      // 内建 TakeCoordinates 单例（SDK 已替你 new 好）
    ├── allTakeCoordinates   // 所有已注册 TakeCoordinates 的数组（含内建）
    ├── snapTargetType / snapTypeList / snapSelectedResults / Snap / cancelSelectedPickObject()
    └── (鼠标命中场景对象后) → 把底层事件 fire 给每个已注册的 TakeCoordinates

Glodon.CIMCube.Controls（导出命名空间，只有 2 个类）
├── class TakeCoordinates    // 坐标拾取/锚点管理器：能 active 后自动累积左键点并派发事件
└── class PickEventHandler   // 轻量事件接收壳（extends EventDispatcher，无取点累积逻辑，很少直接用）
```

要点：

- **一个 TakeCoordinates 实例只做「收拾取事件 + 存点 + 派发事件」，不做任何绘制**：不加点标记、不画线、不显示提示条。视觉全部由你在业务层用 Graphic/Polyline 等自绘（监听事件拿到坐标）。
- 控制器把每一次有效鼠标命中（点到地形/球面/模型图层/矢量对象）打包成底层事件发给**所有已注册** TakeCoordinates；只有处于 active 状态的实例会按左键自动累积点。
- 点的是纯空白天空（无地表无对象）时不产生事件——这是唯一取不到点的情况。

## 二、Quick Start（真实 API）

```js
const D = Glodon.CIMCube.Global.Defines;

// —— 方式一（推荐）：直接用 map 内建单例，无需手动注册 ——
const tc = map.controls.takeCoordinates;

// —— 方式二：自建隔离实例（业务并发时更安全，用完 dispose 归还注册表）——
// const tc = new Glodon.CIMCube.Controls.TakeCoordinates(map.controls);

// 取 3 个点：点满 3 次自动 finish；再允许右键提前结束
tc.on(D.MapActionType.ADD, (e) => {
  console.log('新增点', e.position, 'key=', e.pointKey, '当前全部', e.coordinates);
});
tc.on(D.MapActionType.FINISH, (e) => {
  console.log('取点完成', e.coordinates);   // [{lon,lat,height}, ...]
});
tc.active(3, true);

// 业务层画点标记（视觉需你自己加）：
const G = Glodon.CIMCube.Graphic;
const layer = map.layerManager.addLayer({
  type: D.LayerType.GRAPHIC,          // 'graphic'
  id: "pick-pts", name: "取点标记",
});
tc.on(D.MapActionType.ADD, (e) => {
  // ★ 事件坐标是 {lon,lat,height} 对象，图元 position 要 [lon,lat,height] 数组，必须转换
  const p = e.position;
  layer.addGraphic(new G.PointGraphic({
    position: [p.lon, p.lat, p.height],
    pixelSize: 8, color: "#ffcc00",   // ★ 点样式键是 pixelSize / color，没有 point:{size}
  }));
});
```

## 三、常见最小模板（N 点取点 -> Promise）

```js
function pickPoints(map, count = 1) {
  const D = Glodon.CIMCube.Global.Defines;
  return new Promise((resolve) => {
    const tc = new Glodon.CIMCube.Controls.TakeCoordinates(map.controls);
    tc.on(D.MapActionType.ADD, () => {});   // 需要时在此加 Graphic 标记
    tc.on(D.MapActionType.FINISH, (e) => {
      tc.dispose();                         // 归还：不污染 allTakeCoordinates
      resolve(e.coordinates);
    });
    tc.active(count, true);                 // 点满 count 或右键结束
  });
}
// const pts = await pickPoints(map, 2);   // [{lon,lat,height},...]
```
