# Controls 业务封装模式（画折线 / 悬浮预览 / 锚点拖拽 / 工具按钮）

> 本文件对应 SKILL.md 路由表的「画折线/多边形、悬浮预览、锚点拖拽、工具按钮」行。

> 相关：`references/quick-start.md`（骨架）、`references/events.md`（两套事件通道）、`cimcube-graphic`（把坐标画成点/线/面）

## 6.1 取点画折线/多边形范围（视觉与「工具完成」全在业务层）

```js
const D = Glodon.CIMCube.Global.Defines;
const G = Glodon.CIMCube.Graphic;
const polyLayer = map.layerManager.addLayer({
  type: D.LayerType.GRAPHIC,          // 'graphic'
  id: "pick-poly", name: "取点范围",
});
const tc = map.controls.takeCoordinates;
let graphic = null, pts = [];

function resetPick(needPointCount) {
  pts = [];
  tc.clear();
  tc.active(needPointCount, true);
}
tc.on(D.MapActionType.ADD, (e) => {
  pts.push(e.position);                            // {lon,lat,height} 对象
  // ★ 图元 positions 要 [lon,lat,height] 数组，对象必须先转换
  const ring = pts.map((p) => [p.lon, p.lat, p.height]);
  if (ring.length > 1) ring.push(ring[0]);         // 首尾闭合
  if (graphic) polyLayer.removeGraphic(graphic);   // ★ removeGraphic，不是 remove
  graphic = new G.PolylineGraphic({
    id: "pick-line", name: "范围线",
    positions: ring,
    width: 3, material: "#00ffff",                 // ★ 线样式键是 width / material，没有 polyline:{}
  });
  polyLayer.addGraphic(graphic);                   // ★ addGraphic，不是 add
});
tc.on(D.MapActionType.FINISH, (e) => {
  // 拿到全部坐标，转成 Analysis/Measure 参数或保存
  // 最后 tc.clear(); // 复位监听，避免残留 active
});
```

## 6.2 鼠标悬浮实时预览坐标（走底层 MOUSE_MOVE 通道）

> 注意：派生 `MOVE`（`MapActionType.MOVE`）要求 `isActive && enableMouseMove`，会顺带累积左键点；纯悬浮预览请用底层 `ScreenSpaceEventType.MOUSE_MOVE`（只需 `enableMouseMove`，即 `activeMove()` 置位，不依赖 active）。

```js
const D = Glodon.CIMCube.Global.Defines;
const tc = map.controls.takeCoordinates;
tc.activeMove();                                  // 打开鼠标移动事件流（不会累积点）
tc.on(D.ScreenSpaceEventType.MOUSE_MOVE, (e) => { // 底层通道，不需要 active
  if (e.position) statusDiv.textContent = `${e.position.lon}, ${e.position.lat}, ${e.position.height}`;
});
// 结束时：tc.stopMove();
```

## 6.3 点锚点可拖拽（active + beginMoving + 派生 MOVE）

```js
const D = Glodon.CIMCube.Global.Defines;
const tc = map.controls.takeCoordinates;
tc.active(999, true);                             // 开启取点（右键可结束拖拽，见下）
tc.on(D.MapActionType.ADD, (e) => tc.beginMoving(e.coordinates.length - 1)); // 新点落点后立即进入拖拽
tc.on(D.MapActionType.MOVE, (e) => {
  if (e.movingIndex !== undefined) refreshGraphicVertex(e.movingIndex, e.position); // 同步业务图形顶点
});
// beginMoving(index, true)：右键结束拖拽（内部 stopMove）；手动结束用 tc.stopMove()
```

## 6.4 完成态封装为独立工具按钮（开始/停止）

```js
function startPick() { tc.clear(); tc.active(999, true); }   // 右键结束
function stopPick()  { tc.deactive(); tc.clearPoint(); }
```

## 6.5 组合与排错提示

- 画完要留痕：`FINISH` 里把 `e.coordinates` 转成 Graphic 持久化，并在最后 `tc.clear()`，避免残留 active 影响下一次取点。
- 需要「点中谁」而不是「点在哪」：用底层通道的 `pickedObject`（见 `references/events.md` 通道 B）。
- 取不到点的唯一情形是点到纯空白天空；若偶尔丢点，检查是否误开了 `enabledClickDelay`（双击消抖 200ms）。
- 多实例并发（如同时有两个工具在取点）优先自建 `new TakeCoordinates(map.controls)` 并在结束时 `dispose()`，不要复用内建单例。
