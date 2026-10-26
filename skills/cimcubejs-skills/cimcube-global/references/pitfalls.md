# Global 避坑记录（枚举 / Math 方法黑名单）

> 本文件对应 SKILL.md 路由表的「枚举不存在项与拼写差异、Math 方法黑名单（APIDOC 有但 runtime 未导出）」行。

> 相关文件：枚举清单 → `defines-enums.md`；方法清单 → `math-methods.md`。

## 一、Defines 枚举避坑

1. **高度/贴地枚举不在 Defines**：`heightReference` 的真实枚举是 **`Glodon.CIMCube.Graphic.HeightReference`**（如 `G.HeightReference.CLAMP_TO_GROUND`，值为数字 `NONE 0` / `CLAMP_TO_GROUND 1` / `RELATIVE_TO_GROUND 2` / `CLAMP_TO_TERRAIN 3` / `CLAMP_TO_3D_TILE 5` 等，含 CIMCube 扩展模式，**不要套 `Glodon.CIMCube.Cesium.HeightReference` 的取值**）——详见 cimcube-graphic。
2. **无统一事件/动画/分析/锚点等枚举**：`EventType`/`AnimationType`/`AnalysisType`/`WidgetAnchor`/`ColorPreset`/`LogLevel` 等不在 `Global.Defines`。事件没有统一 `EventType` 枚举：图层事件用 `Defines.LayerEventType.*`（**数值枚举，不能传字符串 `"LOADED"`，应 `Defines.LayerEventType.LOADED`**），取点/底层用 `Defines.ScreenSpaceEventType.*` 与 `Defines.MapActionType.*`（`FINISH` 等是 `MapActionType` 成员）；动画/分析无集中类型枚举，直接 `new Analysis.xxx` / `new Animation.xxx`。控件锚点用 `Defines.Anchor`（runtime 键 `center`/`leftBottom` 等小驼峰字符串）。
3. **枚举值类型分两类**：字符串值枚举（LayerType/ImageryLayerType/GraphicType/Anchor 等）值为小写字符串；number 型（AnimationStatus/ScreenSpaceEventType 等）才是数字。判断用 `DEF.XXX.YYY` 引用，别写死。
4. **APIDOC 拼写与 runtime 不同**：`GeocoderMode`（runtime）在文档写作 `GeoCoderMode`；以 runtime 键为准。

## 二、Math 方法避坑

1. **通用距离/面积 API 不存在于 Math**：两点距离/方位角、面积请用 `cimcube-utils` 的 `Utils.Measure`（有 APIDOC），`Global.Math` 无此能力。
2. **APIDOC 拼写与 runtime 不同**：`caluateMatrix` 首字母**小写 c** 才是 runtime 键（APIDOC 渲染为 `CaluateMatrix`）。
3. **方法级黑名单（APIDOC 列出但 runtime 不导出）**：以下方法在 `Global.Math.html` 中可见，但 `Glodon.CIMCube.Global.Math` **运行时并不导出**，调用会 `undefined`（浏览器实测），**不要使用**：
   `equals` / `getDistance` / `getRayIntersectionToPlane` / `getTransformationFromSRT` / `insidePolygon` / `linePolygonIntersection` / `pointInPolygon` / `packedDepth` / `unpackDepth` / `reverseBezierCurveFn`。
4. **`modelToWorldCoordinates` 签名与文档不同**：入参是**单个扁平数组 `[x,y,z]`**（内部 `Cartesian3.fromArray`），返回**单个 `[lon,lat,height]`**；传 `{x,y,z}` 对象或嵌套数组会得到 `[null,null,null]`（浏览器实测，SDK 源码 `Cartesian3.fromArray(e)` 佐证）。见 `math-methods.md` §2.2。

## 三、Logger 避坑

1. **`Global.Logger` 是容器对象，不是 `Global.Logger.Logger` 类导出**：runtime 实测（浏览器）`Glodon.CIMCube.Global.Logger` 为 `{ logger, LogLevels }`——
   - `Global.Logger.logger`：**预建实例**，自带 12 个日志方法 `debug/trace/log/info/warn/error/assert/count/dir/time/timeEnd`（自有属性）+ `setLevel`（原型）+ `level`（当前级别，数值对应 LogLevels）；
   - `Global.Logger.LogLevels`：7 级枚举 `DEBUG(6)/PERFORMANCE(5)/LOG(4)/INFO(3)/WARNING(2)/ERROR(1)/NONE(0)`；
   - **不要写 `new Global.Logger.Logger()`**（`Logger.Logger` 为 `undefined`）；直接用预建实例 `Global.Logger.logger.info(...)`。
