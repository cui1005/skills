# Utils.Measure — 交互量测（13 种 MeasureMode）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

## 构造签名与 13 种模式

`new U.Measure(map, measureMode [, options])`：

- `measureMode`（必填）：`DEF.MeasureMode.X` 枚举，决定交互与显示（**不要用字符串如 `"distance"`，无此形态**）。
- `options`（可选，`{}` 缺省）：不同模式对应不同 Options 结构（见下表），字段级配置由 SDK 决定。

| MeasureMode（值） | 测量内容 | 对应 options 类型 |
|---|---|---|
| `Coordinate` (1) | 坐标量测：拾取点读取经纬高 | CoordinateOptions |
| `Angle` (2) | 夹角（两线方向夹角） | AngleOptions |
| `Azimuth` (3) | 方位角 | AzimuthOptions |
| `Space_Distance` (4) | 空间距离（三维直线） | SpaceDistanceOptions |
| `Ground_Distance` (5) | 贴地距离（沿地形） | GroundDistanceOptions |
| `Triangulation` (6) | 三角测量（详见下方） | TriangulationOptions |
| `Space_Area` (7) | 空间面积 | SpaceAreaOptions |
| `Ground_Area` (8) | 贴地面积（沿地形） | GroundAreaOptions |
| `Project_Area` (9) | 投影面积（水平面） | ProjectedAreaOptions |
| `Slope` (12) | 坡度 | SlopeOptions |
| `Model_Distance` (13) | 贴模型距离（沿模型表面） | ModelDistanceOptions |
| `Volume` (14) | 体积测量 | VolumeOptions |
| `Project_Distance` (15) | 投影距离（水平投影） | ProjectDistanceOptions |

> 相关枚举也在 `cimcube-global` §1.4：`MeasureShowResult`（结果展示时机 always/never/trigger_hover/trigger_click）、`DatumMode`（体积基准面）、`LengthUnit`/`AreaUnit`（单位）。

## 交互流程与事件

`activate()` 后进入测量（防重入：已激活会 console 提示并忽略）：左键拾取点，移动鼠标实时预览（内部相当于 `updatePosition`），每完成一个拾取点派发 `ADD`；右键点击或调用 `completion()`（二者等价）结束本次测量并派发 `FINISH`；测量中可 `undo()` 单步撤销最近一个点。`deactivate()` 退出，`dispose()` 释放（内部会 deactivate + 清除标注）。

```js
const m = new U.Measure(map, DEF.MeasureMode.Ground_Area, {});
m.on(DEF.MapActionType.ADD,    (e) => console.log("已添加一个点", e));
m.on(DEF.MapActionType.CHANGED, (e) => console.log("测量数据变化", e));
m.on(DEF.MapActionType.FINISH, (e) => {
  // e = { mode: MeasureMode, graphic: BaseGraphic } —— 本次绘制几何图层（Polyline/Polygon）
  console.log("结束，mode =", e.mode, "graphic =", e.graphic);
});
// 可选：原始左/右键事件（EventProperty，addEventListener 注册）
m.leftButtonClicked.addEventListener((pos) => console.log("鼠标左键拾取", pos));
m.rightButtonClicked.addEventListener(() => console.log("鼠标右键"));
m.activate();
```

> 结果数值随界面标注显示；`FINISH` 载荷只含 `{mode, graphic}`。需要把数值入库时，对 FINISH 返回的 `graphic`（几何引用）用 `MeasureUtils.MeasureFormula`（见 `MeasureUtils.md`）重算，或监听展示结果（`MeasureShowResult` 相关配置）。

## 方法速查

| 方法 | 说明 | 备注 |
|---|---|---|
| `activate() / deactivate()` | 激活 / 退出量测 | 激活带防重入 |
| `completion()` | 完成量测（等同于鼠标右键点击） | 程序化结束时用 |
| `setPosition({lon,lat,height})` | 加一个测量点（等同于一次左键拾取） | 程序化取点 |
| `updatePosition({lon,lat,height})` | 更新悬停点（等同于 mouseMove） | 模拟移动 |
| `undo()` | 单步撤销最近一个拾取点 | |
| `clearAll()` | 清除全部量测标注 | |
| `getMeasureLayer()` | 获取量测图层（`Layer.GraphicLayer`） | |
| `isActive` / `measureMode` | 只读成员：是否激活 / 当前模式 | |
| `delete(id)` / `get(id)` / `show(id)` / `hide(id)` | 移除 / 获取 / 显示 / 隐藏绘制结果 | **仅三角测量模式** |
| `setTriangulationType(type)` | 设三角测量方式，重新激活生效 | **仅三角测量模式** |
| `setStyle(options)` | 设量测样式，重新激活生效 | **仅三角测量模式**，只收样式相关参数 |
| `setLayer(layer)` | 设测量的图层 | **仅体积测量**（只支持 Tiles3DLayer） |
| `setDatum(opts)` | 设测量基准面 | **仅体积测量**，如 `{ mode: "customPlane", altitude: 234.232 }`；`DatumMode` 另有 lowestPointPlane / highestPointPlane / middlePointPlane |
| `setVolume(enable, volumeColor)` | 显隐测量范围 + 颜色 | **仅体积测量** |
| `restore(positions)` | 恢复测量结果 | `positions = [[lon,lat,height], …]` |
| `on / off` | 监听 / 解绑事件 | `on == addEventListener`，同一事件只保留一个回调（重复注册 console 提示） |

## 三角测量（Triangulation）

对建筑墙体做"正视 / 侧视两视图定位量测"的场景：`TriangulationType.DEFAULT`（默认，水平+垂直同时支持）/ `HORIZONTAL` / `VERTICAL`，通过 `setTriangulationType(DEF.TriangulationType.HORIZONTAL)` 切换后**重新 activate** 生效。该模式会生成可管理的绘制结果（`get(id)/delete(id)/show(id)/hide(id)`）。

## 体积测量（Volume）

`MeasureMode.Volume`：在 `setLayer(Tiles3DLayer)` 指定的模型层上圈定底面并拉高形成体积（配合基准面）。流程示例：

```js
const mv = new U.Measure(map, DEF.MeasureMode.Volume, {});
mv.setLayer(tiles3dLayer);                 // 只支持 Tiles3DLayer，否则 warn
mv.setVolume(true, Cesium.Color.YELLOW);   // 显示量测范围并着色
mv.setDatum({ mode: "customPlane", altitude: 100 }); // 基准面：自定义高程面
mv.activate();
// 圈定完成后读取体积：监听 FINISH，或用 volumeGraphic 相关几何自行计算
mv.dispose();
```

> 与 `Analysis.CutFill`（填挖方）的区别：CutFill 关联设计面/地形算填挖量，`MeasureMode.Volume` / `Utils.VolumeCalculation` 只按几何做体积测算。

## 性能与三戒

| 工具 | 开销 | 建议 |
|---|---|---|
| Measure | 低~中（挂屏幕事件 + 实时标注） | 一次只激活 1 个；用完 `dispose()`（释放会清标注并退深度限制） |

> 🌟 **三戒**：① 同一时刻只激活 1 个测量工具；② 交互结束立即 `dispose`（非仅 `deactivate`）；③ 单位换算先确认输入基准（长度米 / 面积平方米，见 `MeasureUtils.md`）。

## 常见模式

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
m.on(DEF.MapActionType.FINISH, (e) => {
  // e.graphic 上可继续做业务处理；或把点集交给 MeasureFormula 重算
  console.log("finished", e.mode);
});
m.setPosition({ lon: 116.40, lat: 39.90, height: 30 }); // 等价一次左键拾取
m.setPosition({ lon: 116.41, lat: 39.91, height: 80 });
m.completion();                 // 等价右键，结束当前量测 -> FINISH
m.dispose();
```
