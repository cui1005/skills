# Compass / CompassWidget 罗盘控件

> 来源：CIMCube SDK API_DOC —— `Glodon.CIMCube.Widgets.CompassWidget.html`、`Glodon.CIMCube.Widgets.Compass.html`（JSDoc 4.0.5，2026-08-31）。
> 本文件只写罗盘；样式锚点清单见 `styles.md`，导出现实/句柄取法见 `BaseWidget.md`。

## 1. 两个类，别混用

| 类 | 导出 | 说明 |
|---|---|---|
| `Widgets.CompassWidget` | ✅ 导出 | 业务唯一入口：`new W.CompassWidget().addToMap(map)`。**mixin 包装**，无 `enabled/show/hide` |
| `Widgets.Compass` | ⚠️ 未导出 | 罗盘 UI 本体，`extends BaseWidget`；由 `CompassWidget` 内部实例化，**无对外访问器** |

## 2. CompassWidget（API_DOC 事实）

```js
new CompassWidget()          // 无构造参数
```

| 方法 | 参数 | 说明 |
|---|---|---|
| `addToMap(map)` | `map: Glodon.CIMCube` | 将控件添加至 CIMCube 中 |

- API_DOC 中 **没有** options、成员、事件——`new CompassWidget(options)` 传参无效，也不会被消费。
- `addToMap` **无返回值**（mixin 包装，内部实例不对外暴露），因此**没有句柄可调** `enabled/show/hide`。

```js
const W = Glodon.CIMCube.Widgets;
new W.CompassWidget().addToMap(map);   // 挂载即生效，无需保存返回值
```

## 3. 交互与运行时行为

罗盘本体（`Compass`）提供的鼠标交互：拖**外圈**旋转（改 heading）、拖**内圈陀螺**俯仰（改 pitch）、**双击**回正北、`+ / -` 缩放按钮。这些都是控件内部行为，**没有对外 API**：想用代码改视角/回正北请走 `map.camera`（见 `cimcube-camera`）。

数据来源是 `Utils.CompassService`（Service 层属 **cimcube-utils**），业务侧无需接触。

## 4. 显隐 / 移除：只能操作根 DOM

`CompassWidget` 无返回值、无 `enabled/show/hide`，所以显隐与卸载一律走根元素 `.cimcube-compass`：

```js
const el = document.querySelector(".cimcube-compass");
el.style.display = "none";   // 隐藏
el.style.display = "";       // 恢复
el.remove();                 // 彻底移除（不要再次 addToMap，避免重复挂载）
```

> 需要"跟随相机/批量显隐浮层"这类统一控制，见 `patterns.md` 的模式 A/C（罗盘与搜索框等一并切 `display`）。

## 5. 内部类 Compass（未导出，仅供理解）

`new Compass()` 在 API_DOC 中的记录：

- **Extends**：`Glodon.CIMCube.Widgets.BaseWidget`
- **Members**：`enabled : Boolean` —— 是否启用（Inherited From + Overrides `BaseWidget#enabled`）
- **Methods**：`show()` 显示控件、`hide()` 隐藏控件（均 Inherited From + Overrides `BaseWidget#show/hide`）

⚠️ 这些成员**只对内部类 `Compass` 有效**（`enabled` 挂/卸 DOM，`show/hide` 只切 `visibility`）；`CompassWidget` 不继承 `BaseWidget`，**拿不到**它们。业务也 `new` 不出 `Compass`（`new W.Compass()` 为 undefined）。

## 6. 样式锚点

`Compass.css`（已聚合进 `Widgets.css`，不要单独引）：`.cimcube-compass`、`.cimcube-out-ring`、`.cimcube-gyro`、`.cimcube-zoom-controller`。

```css
.cimcube-compass { right: 180px; }   /* 换位：放自有样式表，且置于 SDK CSS 之后 */
```

## 7. 坑点

| 误用 | 正确做法 |
|---|---|
| `new W.Compass(...)` | ❌ 内部类未导出；用 `new W.CompassWidget().addToMap(map)` |
| `const c = new W.CompassWidget().addToMap(map); c.hide()` | ❌ `addToMap` 无返回值；直接操作 `.cimcube-compass` |
| `new W.CompassWidget({ position: "left" })` | ❌ 无 options；改位置用 CSS 覆盖 `.cimcube-compass` |
| 重复 `addToMap` 罗盘 | ⚠️ 勿重复（会抛 `Cannot redefine property`，机制见 **references/pitfalls.md**「单例与句柄」条）；挂前判 `.cimcube-compass` 已存在则跳过，卸载用 `remove()` |
| 代码里想"回正北" | ❌ 罗盘无对外方法；用 `map.camera`（`cimcube-camera`，如 `setView({ heading: 0 })` / `flyTo`） |
