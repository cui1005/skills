# Utils 剖切族 — ClipBase / BoxClip / SceneClip / ModelClip / TerrainClip

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

## 相关枚举

```js
const D = Glodon.CIMCube.Global.Defines;
D.ModelClipMode;      // {VERTICAL:0, HORIZONTAL:1}   剖切面方向（SceneClip/ModelClip 用）
```

## 一、共同机制与 ClipBase

- 剖切实质：给支持裁剪的目标（实现 **clippableTarget** 接口：`setClippingPlanes(planes)` + `boundingVolumeOrigin` 等）设置 `ClippingPlaneCollection`。图层侧通常在 `Layer`/3DTiles 上原生支持，所以这些工具多传 **map/图层**，而不是 Primitive。
- `ClipBase`（运行时：`Utils.ClipBase`）是 ModelClip/SceneClip 等的父类：
  - 静态常量：`ClipBase.EVT_POS_CONFIRMED = 'poscomfirm'`、`ClipBase.EVT_READY_TO_CLIP = 'rtclip'`。
  - 内部 fire 载荷：`poscomfirm → {type:'poscomfirm', positionInWorld}`；`rtclip → {type:'rtclip', clipTarget}`。
  - 实例方法：`on(type, cb, thisArg?)` / `off(type, cb)` / `dispose()`。

## 二、BoxClip — 盒体剖切（显示盒内 / 挖盒看内部）

| 项 | 内容 |
|---|---|
| 构造 | `new U.BoxClip(map, { labelColor?, fontSize?, fontColor? })`——这些只控制"选择模型"提示文字样式 |
| `clip(target)` | **LAYER 模式**：对单个 target（图层等 clippableTarget）执行盒剖切并挂可拖盒 UI |
| `clipEx(posA, posB, height, offset = 0, arrTarget?)` | **OVERALL 模式**：`posA/posB` 为盒底面两个对角点世界坐标（Cartesian3），`height` 盒高（米），`offset` 盒底距地表高度（默认 0 = 贴地），`arrTarget` 为图层数组，**不传则作用于全部已就绪的受支持图层** |
| `pickOn()` / `pickOff()` | 开启/关闭"选择模型"鼠标跟随提示 + 点击拾取图层（点击命中后自动 `clear()+clip(该图层)`）；APIDOC 明确 **只有 LAYER 模式才支持拾取图层** |
| `getState()` / `setState(state)` | 存取 UI 高亮色/盒 min/max/mode/param，可做保存恢复 |
| `clear()` | 复位并释放当前盒剖切（含恢复相机输入、移除 clip） |
| `dispose()` | 销毁（clear + UI destroy + dispatcher dispose） |
| 事件 | `on/off`；静态 `BoxClip.BoxClipEvent = {MOUSE_ENTER:'mouse_enter', MOUSE_LEAVE:'mouse_leave'}`（盒 UI 悬停进出） |

```js
// 全局盒剖切（默认剖全部支持图层）：底面两点 + 高 80m、离地 0
const box = new U.BoxClip(map, { fontColor: '#fff', labelColor: '#20262F' });
box.clipEx(
  Glodon.CIMCube.Global.Math.Cartesian3.fromDegrees(116.35, 39.9),
  Glodon.CIMCube.Global.Math.Cartesian3.fromDegrees(116.36, 39.91),
  80, 0
);
// 之后可用鼠标拖盒 UI 调整；结束：box.clear(); box.dispose();
```

> ⚠️ 臆造（不可用）：`new U.BoxClip(map, { targetLayers, box:{min,max}, invert }) + boxClip.apply()` 不存在。正确为 `clip(target)`（LAYER）/ `clipEx(posA,posB,height,offset?,targets?)`（OVERALL）；**无 invert**（裁剪方向由模式决定）。

## 三、SceneClip — 场景剖切（平面剖切多对象 / 含地形与开挖）

| 项 | 内容 |
|---|---|
| 构造 | `new U.SceneClip(map, options)`，options 可选：`clipMode`（`D.ModelClipMode.VERTICAL/HORIZONTAL`）、`targets`（Array，不传剖全部支持对象）、`clipPlaneUI`（自定义剖切面 UI，默认内置 UI）、`length`（剖切面长度，'auto'/'infinity' 等）、`includeTerrain`（默认 false，true 时切到地形/挖坑需配合 excavate）、`excavate`（`{offset, texture:{facade,bottom}}` 开挖设置）、`edgeColor/edgeWidth`（剖切边线）、`planeColor/planeAlpha`（剖面填充）、`planeBorderColor/planeBorderAlpha/planeBorderWidth`、`axisColor/axisAlpha`、`debug_clipOnly`（仅供调试） |
| 交互式 | `activate()` 开启剖切（显示剖切面与手柄，可拖动/旋转），`deactivate()` 关闭并清理 |
| 编程式 | `clip(posInWC_a: Cartesian3, posInWC_b: Cartesian3, mode?: D.ModelClipMode)` 两点定义剖切位置与朝向直接剖 |
| 属性 | `UI`（只读，内置剖切面 UI 对象）、`clipMode`、`planeSize`、`isActive`（只读） |
| 其它方法 | `setPlaneColorAndAlpha(...)` / `setPlaneBorderColorAndAlpha(...)`、`translateTarget(defaultValue, relative)`（移动被剖对象）、`clear()`、`dispose()` |
| 继承 | extends ClipBase（事件/on/off 见 §一）；**activate() 期间可结合 `ClipBase.EVT_POS_CONFIRMED` 拿用户确认的位置** |

```js
const sc = new U.SceneClip(map, {
  clipMode: D.ModelClipMode.VERTICAL,
  targets: [s3mLayer],          // 只剖这层；不传 = 全部
  edgeColor: '#00ff88',
  includeTerrain: true,          // 需要一起剖地形时开启
});
sc.activate();                    // 交互拖剖切面（示例：用户拖好后拿状态）
sc.on(U.ClipBase.EVT_POS_CONFIRMED, (e) => console.log('剖切位置', e.positionInWorld));
// sc.deactivate();               // 停止；或 sc.clear(); sc.dispose();
```

## 四、ModelClip — 模型剖切（单体模型切面）

| 项 | 内容 |
|---|---|
| 构造 | `new U.ModelClip(map, { clipMode?: D.ModelClipMode, edgeColor?, edgeWidth?, layer? })`；`layer` 指定被剖切单体（`Cesium3DTileset / Entity` 或 BaseLayer），**不指定时程序根据用户绘制的剖切线自动找最近模型** |
| 交互式 | `activate()`：提示并让用户在地图上点两下绘制剖切线（生成切面）；`deactivate()`：停止 |
| 编程式 | `clip(posInWC_a: Cartesian3, posInWC_b: Cartesian3, mode?: D.ModelClipMode, target?: BaseLayer)` |
| 属性 | `clipMode`、`isActive`（只读） |
| 其它 | `clear()`、`dispose()`；extends ClipBase（事件见 §一） |

## 五、TerrainClip — 地形开挖基础件（⚠️ 无独立 APIDOC 页，进阶用法）

与 `ModelExcavation` 的区别：TerrainClip 针对**地形**（globe 材质挖洞/贴面）。API（原型实测）：`depth` / `bottomTextureUrl` / `facadeTextureUrl` / `showTexture` / `regions` / `mode` 属性；`setDepth/setBottomTexture/setFacadeTexture`；`addRegion(region, ...)/removeRegion(...)`、`doAnalysis()`、`getResult()`（返回被裁剪到的 3DTiles 图层名/列表）、`clear()/dispose()`。
`SceneClip` 开启 `includeTerrain:true` 并配置 `excavate` 时内部即使用 TerrainClip 完成地形侧的开挖补面，因此**优先用 SceneClip 的 excavate 参数**，直接构造 TerrainClip 属于进阶用法。
