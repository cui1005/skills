---
name: cimcube-camera
description: "CIMCube 相机控制：视角/姿态读写、定位 setView、飞行动画 flyTo、环绕旋转、屏幕拾取椭球坐标、读取当前视角、截图、相机事件。当用户需要让相机飞到某个坐标或图层、设置/读取视角、旋转地球、getCaptureImg 截图或读取相机信息时激活。"
---

# CIMCube 相机控制（Camera · 单技能路由）

> 类 `Glodon.CIMCube.Camera`
> **入口**：`map.camera`（CIMCube 实例自动创建；APIDOC 虽声明构造器 `new Camera(map)`，实际业务统一用 `map.camera`）
> **文档与运行时口径**：本技能参数/单位/默认值均已对照 APIDOC 类页与 `cimcube.js` 实现逐项核实
> **定位**：本 SKILL.md 只做**索引路由**。完整 options 表、可运行示例与坑点全部外置于同目录 `references/<方法族>.md`。**写某个方法的完整代码前，先读对应文件**，不要凭记忆重写示例。

## 跨域约定（适用于全部 Camera 方法）

1. **单位三套，最易混**：
   - 位置 `lon/lat/height`：经/纬度=**度**、高度=**米**（字段固定 `lon/lat/height`，**不是** `x/y/z`、`lng/lat`）。
   - 姿态 `heading/pitch/roll` 与 `orientation`、`getCameraInfo` 读值：**弧度 rad**。
   - `fov` 视野角：**度**（唯一角度制相机成员）。
   - `flyToBoundingSphere.options.offset`：heading/pitch=**度**、range=**米**（内部转弧度，⚠️ 与 `orientation` 弧度制**不同**，最易踩坑）。
2. **姿态成员只读**：`heading/pitch/roll` 无 setter——改姿态只能走 `setView`/`flyTo`/`lookAt` 的 orientation。**不存在** `camera.x/y/z` 属性。
3. **定位/飞行入口**：按坐标/图层飞行一律用 `map.camera.*`；`map.flyTo` 是主类方法，只吃实体/数据源（传经纬度不动相机）。
4. **截图前提**：`getCaptureImg()` 需 WebGL 上下文开启 `preserveDrawingBuffer: true`（构造时经 `params.contextOptions.webgl` 注入，运行中无法开启）。
5. 公共初始化与组合模式见 `references/patterns.md`；完整误用对照见 `references/pitfalls.md`。

## 方法族路由

| 方法 / 成员 | 用途 | 要点 | 写码前必读 |
|---|---|---|---|
| `flyTo(options)` | 平滑飞行（**单对象参数**） | `{target, flyToPoint, orientation, duration, …}`；target 支持图层 / 经纬高 / Cartesian3 / Rectangle | `references/flyTo.md` |
| `setView(options)` | 无动画瞬间定位 | `{destination, orientation}`；destination 支持经纬高 / Rectangle / Cartesian3；orientation 弧度 | `references/positioning.md` |
| `flyToBoundingSphere(options)` | 按包围球飞行定位 | `center+radius` 或 `bbox`；`offset.heading/pitch` 为**度**、range 米；duration 默认 3 | `references/positioning.md` |
| `lookAt(target, offset)` | 面向某目标（相机不动） | target=Cartesian3 / {lon,lat,height}；offset=Cartesian3 / HPR | `references/positioning.md` |
| `cancelFly()` / `zoomIn(amount?)` / `zoomOut(amount?)` | 取消飞行 / 沿视线进退 | zoomIn/zoomOut amount 默认 1000 米 | `references/flight-control.md` |
| `startRotate(center?, duration=60, clockWise=true, loop=true)` / `stopRotate()` | 相机环绕自转（地球 / 绕点） | center 为 Cartographic，按**弧度**解释（建议 `Cartographic.fromDegrees`） | `references/flight-control.md` |
| `getCameraInfo(includeCameraCopy?)` | 读相机状态（可生成恢复视角代码） | 返回 lon/lat/height + ECEF x/y/z + 姿态(弧度)；`true` 附 `cameraCopy` | `references/read-view.md` |
| `getCurrentViewInfo()` | 读视角 + 屏幕中心落点 | `{orientation, location, positionWC, center?}` | `references/read-view.md` |
| `print()` | 调试打印相机信息 | 等价 `getCameraInfo(true)` 前两行 | `references/read-view.md` |
| `getCaptureImg()` | 截图（同步返回 dataURL） | 无参、同步 `{image,width,height}`；需 `preserveDrawingBuffer` | `references/read-view.md` |
| 属性成员 `position / positionCartographic / heading·pitch·roll / fov / mvtZoom / decreaseRequestingWhileFlying / trackedModel` | 相机属性读写 | `heading/pitch/roll` **只读**；`trackedModel` 赋值即跟踪模型 | `references/members.md` |
| 事件 `changed / moveStart / moveEnd` | 相机事件（内核 Event 对象） | `addEventListener` / `removeEventListener`（**非** `on("…")`） | `references/members.md` |
| `dispose()` | 停止自转 + 取消飞行并释放相机 | 一般随 `map.dispose()` | `references/members.md` |
| 常见模式 / 实用提示 | 初始化视角、加载后飞行、截图、书签、环绕展示 | 5 个可直接套用的组合 | `references/patterns.md` |
| ⚠️ 误用对照表 | 易错写法 → 正确口径 | 12 条常见坑 | `references/pitfalls.md` |

## See Also

- **cimcube-setup** — CIMCube 构造参数、`center` 初始视角约定、`params`/contextOptions 注入
- **cimcube-global** — `Global.Math` 度/弧度转换与坐标工具、`Global.Defines` 事件名等常量
- **cimcube-layer** — `LayerManager.getLayer` 取图层实例作为 `flyTo.target`；图层自带 `flyTo()` 时机
- **cimcube-widgets** — Compass（罗盘）/CameraRoam（相机漫游面板）等 UI 相机控制（cimcube-widgets/references/CameraRoam.md）
- **cimcube-animation** — CameraRoaming 路径漫游等效果层相机行为

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
