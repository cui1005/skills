# BaseWidget 基类语义 + Widgets 导出现实

> 基类语义与导出现实对照。

## 1. BaseWidget：enabled / show / hide / addTo

`StatusBar`、`DistanceLegend`、`Compass` 运行时都继承它。`BaseWidget` 本身**未导出**（`new W.BaseWidget()` 为 undefined），仅供理解行为：

```js
// BaseWidget 生命周期（代码即文档）：
const sb = new W.StatusBar();
sb.addToMap(map);               // 真正挂载（内部 append wrapper 到地图容器并启动服务），返回值即控件句柄
// enabled 置 true/false → 从地图容器添加/移除 DOM + bind/unbind 事件
sb.enabled = false;             // 整体从 DOM 移除（真正停用，StatusBar 会停止取数）
sb.enabled = true;              // 重新挂回
sb.show();                      // style.visibility='visible'（不触发服务启停）
sb.hide();                      // style.visibility='hidden'
```

- `show()/hide()` 只改 `visibility`（布局还在，服务仍跑）；`enabled` 才控制是否存在于 DOM。
- `addTo(...)` 是底层挂载；业务一般用便捷的 `addToMap(map)`。⚠️ **`addToMap/addTo` 一律返回 `undefined`**——句柄请**挂前持实例** `const w = new X(...); w.addToMap(map);` 再用 `w`（见 §3）。
- 反例：`CompassWidget / GeocoderWidget` 是**无继承的 mixin 包装**，**不是** `extends BaseWidget`，所以它们没有 `enabled`/`show`/`hide`。

## 2. 导出现实：全局仅 11 类

`Glodon.CIMCube.Widgets` 全局对象实际只导出 **11 个类**：

```
CompassWidget  GeocoderWidget  StatusBar  DistanceLegend
CameraRoam     GroundRoam
MapLegendWidget  MapSplitWidget  DragModel
```

> `LayerControlWidget` 虽在这 11 个导出类里，但**不是业务组件**：只由初始化参数 `params.layerControl: true` 开启（见 `cimcube-setup/references/constructor-options.md`），业务不 `new`、也不取句柄。

**`new W.xxx` 为 undefined（有 API_DOC 页但未挂命名空间）的内部类/遗留：**

| 类 | API_DOC 页 | 真实可达途径 |
|---|---|---|
| `BaseWidget` | BaseWidget.html | 仅内部基类，供理解 `enabled`/`show`/`hide` |
| `Compass` | Compass.html | `CompassWidget` 挂载后内部实例化，**无对外访问器** |
| `Geocoder` | Geocoder.html | 仅 `GeocoderWidget` 内部实例化，**无访问器** |
| `LayerControl` | LayerControl.html | 由初始化参数 `params.layerControl` 显隐的自动挂载面板内部实例化，**无对外访问器**（业务不 `new`、也不取句柄） |
| `DraggableDialog` | DraggableDialog.html | `CameraRoam`/`GroundRoam` 的父类，**无访问器** |
| `MapLegend` / `MapSplit` | — | `MapLegendWidget`/`MapSplitWidget` 的内部实现；`MapSplit` 引擎见 cimcube-utils（references/MapSplit.md） |

> ⚠️ 压缩代码里存在内部函数 `compassMixin / geoCoderMixin`（被 `CompassWidget/GeocoderWidget` 内部调用），但那**不是**公开/文档化 API，勿 import 使用。`Widgets.CompassMixin / GeocoderMixin / LayerControlMixin` 均不存在。

## 3. 取控件句柄的正确方式

> ⚠️ 内部类实例只挂在**引擎内核原生对象**上，而本技能**不暴露引擎内核对象及其下级接口**；业务代码一律不要去上面取，改按下面两种方式拿句柄。

1. **挂前持实例**（推荐）：`StatusBar` / `DistanceLegend` / `MapSplitWidget` / `MapLegendWidget` 等 `addToMap(map)`（或 `addTo(map)`）**返回 `undefined`**，务必 `const w = new X(...); w.addToMap(map);` 先把实例 `w` 存好，再用 `w.show()/hide()/enabled/dispose()`。
2. **操作根 DOM**：mixin 包装类（`CompassWidget` / `GeocoderWidget`）`addToMap` 无返回值，按 `references/styles.md` 里的 DOM 锚点（`.cimcube-compass` / `.cimcube-geocoderContainer`）直接改 `visibility/display` 或 `remove()`。由初始化参数 `params.layerControl` 开启的图层管理面板根节点是外层 `.cimcube-widget-container`（DOM 锚点见 `references/styles.md` §2）。

CIMCube 自身**唯一**提供的访问器是 `map.mapLegend`（由 `MapLegendWidget(options).addTo(map)` 注册，只读），其他控件均无同名访问器；`Geocoder` 与 `DraggableDialog` 无访问器。
