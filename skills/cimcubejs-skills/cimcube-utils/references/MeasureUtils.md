# Utils.MeasureUtils — 测量公式库 + 量测辅助（纯静态）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

`Glodon.CIMCube.Utils.MeasureUtils` 是**命名空间容器**，含两个静态类：`MeasureFormula`（计算）与 `MeasurementUtils`（辅助/单位换算）。

## 一、MeasureUtils.MeasureFormula — 测量公式库（静态）

两个纯函数，对已有点位/几何做**非交互**计算。`geometry` 为 SDK 几何对象（面 / 线，可直接传测量绘制的 Graphic 引用）。`measureMode` 决定算法并校验：**传入不支持的枚举会抛错**。

```js
const F = U.MeasureUtils.MeasureFormula;
// 距离：Space_Distance（默认）/ Model_Distance / Project_Distance / Ground_Distance
const d = F.calculateDistance(polylineGeom, DEF.MeasureMode.Space_Distance);   // Number 或 {…}
// 贴地/贴模型算法需要 map（Ground_Distance / Model_Distance 必须传第三个参 map）
const gd = F.calculateDistance(lineGeom, DEF.MeasureMode.Ground_Distance, map);
// 面积：Space_Area（默认）/ Project_Area / Ground_Area（Ground_Area 需要 map 贴地）
const a = F.calculateArea(polygonGeom, DEF.MeasureMode.Space_Area);            // Number 或 Promise<Number>
const ga = F.calculateArea(polygonGeom, DEF.MeasureMode.Ground_Area, map);
```

> 支持矩阵（运行时已核对）：distance ∈ {Space_Distance, Model_Distance, Project_Distance, Ground_Distance}；area ∈ {Ground_Area, Project_Area, Space_Area}。其余模式一律 `throw "measure mode is not supported!"`。贴地计算（Ground_*）需要传入 `map` 结合地形；空间/投影计算可不传。

## 二、MeasureUtils.MeasurementUtils — 量测辅助（静态）

APIDOC 仅展示 1 个方法，运行时另含 4 个单位换算方法（下方代码已实测语义，可放心使用）。

```js
const MT = U.MeasureUtils.MeasurementUtils;
// 1) 屏幕坐标拾取世界坐标（修复 depthTestAgainstTerrain=false 时 pickPosition 返回负高程）
//    objectsToExclude：排除对象，单个 或 数组（pick 命中被排除对象时退化到 globe.pick 取椭球面点）
//    heightThreshold：默认 -100，拾取结果高度低于该阈值同样回退 globe.pick
const cart3 = MT.pickPositionByMousePosition(map.scene, windowPos, [graphic], -100); // 第 1 参为场景对象
// 2) 长度换算：输入单位为"米"
MT.lengthUnitConversion(1234.5, DEF.LengthUnit.KM);      // -> 1.2345
MT.getLengthUnitString(DEF.LengthUnit.M);                // -> "m"（mm/cm/km 同理）
// 3) 面积换算：输入单位为"平方米"
MT.areaUnitConversion(234567, DEF.AreaUnit.S_HA);        // -> 23.4567（公顷）
MT.areaUnitConversion(234567, DEF.AreaUnit.S_MU);        // -> 351.85（亩：*15/10000）
MT.getAreaUnitString(DEF.AreaUnit.S_MM);                 // -> "mm²"（另有 c㎡/㎡/km²/公顷/亩）
```

> 单位换算先确认输入基准：长度 = 米、面积 = 平方米。纯函数开销极低，可高频同步调用，重复输入参数可缓存。
