# StatusBar 状态栏

## 1. StatusBar（推荐）

`new StatusBar([options]).addToMap(map)`。options（默认值已按运行时确认）：

```js
const sb = new W.StatusBar({   // ★ addToMap 返回 undefined，句柄用 sb 本身
  coordinate: true,          // 显示 纬度/经度 + 高程，默认 true
  azimuth:    true,          // 方位角°，默认 true
  pitch:      true,          // 俯仰角°，默认 true
  fps:        true,          // FPS / 帧耗时 ms，默认 true
  degreePrecision: 6,        // 坐标小数位，默认 6（Number）
  heightPrecision:  2,       // 高程小数位，默认 2（Number）
});
sb.addToMap(map);
```

> ⚠️ APIDOC 表格把 `degreePrecision / heightPrecision` 类型误标为 Boolean；运行时取值是 **Number**（默认 6 / 2），按数字传。

显示规则（来自内部 `_updateContent`，数据源自 `Utils.StatusBarService`）：

- 坐标：`{abs(纬度)toFixed}N/S  {abs(经度)toFixed}E/W  {高程}m`；
- 附带 `方位角: 整数°`、`俯仰角: 整数°`、`{fps}FPS {ms}ms`，各自受 `coordinate/azimuth/pitch/fps` 开关控制。

**单例**：重复 `addToMap` 会抛 `Cannot redefine property`、`map.statusBar` 访问器不存在、`addToMap` 返回 `undefined`（机制与防重见 **references/pitfalls.md**「单例与句柄」条）；句柄用上面的 `sb` 或根 DOM `.cimcube-status-bar`。DOM：`.cimcube-status-bar` > `.cimcube-location`（`StatusBar.css`）。取数在 `enabled = true` 期间持续，`hide()` 只遮不省电——要彻底停用把 `enabled = false`。

## 2. 样式

`Resource/css/StatusBar.css`：`.cimcube-status-bar` > `.cimcube-location`。覆盖示例：`.cimcube-status-bar { background: rgba(13,71,161,.9) !important; }`。
