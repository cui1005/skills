# MapSplitWidget 卷帘对比（+ 内部 MapSplit）

## 1. 构造与选项

```js
const msw = new W.MapSplitWidget(map.layerManager, {
  mode: DEF.ViewPortModeType.HORIZONTAL,   // 'horizontal'(默认) 左右对比 | 'vertical' 上下对比
  splitPosition: 480,                       // Number 可选：分隔位置（像素）；缺省=屏宽/2（horizontal）或屏高/2（vertical）
  viewPortLayersVisible: [                  // Array 可选：卷帘生效图层（控制左右/上下两侧各自的可见性）
    { id: "layerA", visible: true },
    { id: "layerB", visible: true },
  ],
}).addToMap(map);
```

方向语义（已按运行时逐段实证）：`MapSplit` 内部按 `x > splitPosition`（**HORIZONTAL**，左右分屏，分隔条为全高竖线，拖动沿 X）还是 `y > splitPosition`（**VERTICAL**，上下分屏，全宽横线，拖动沿 Y）裁剪可见侧。

## 2. 交互与 DOM

- 生成整屏覆盖容器（右/下附 `.cimcube-mapsplit-switch` 方向切换钮，两个钮 `.cimcube-mapsplit-button-horizontal` / `-vertical`，选中容器加 `.cimcube-mapsplit-switch-selected`）。
- 分隔条 `.cimcube-mapsplit-slider` + 方向类 `.cimcube-mapsplit-slider-horizontal`（4px×100% 竖线）/ `.cimcube-mapsplit-slider-vertical`（100%×4px 横线），拖动即时 `setSplitPosition`。
- CSS：`Resource/css/MapSplit.css`。

## 3. 运行时控制实例

```js
const ms = msw.getMapSplit();            // 内部 MapSplit（MapSplitWidget 暴露的唯一命令入口）
ms.begin();                              // 启动对比（addToMap 已自动 begin）
ms.stop();                               // 关闭卷帘
ms.setSplitPosition(300);                // 程序设置分隔位置(px)
ms.getSplitPosition();                   // 读取当前分隔位置
ms.dispose();
```

> 底层 `MapSplit` 就是 `cimcube-utils` 的 `Utils.MapSplit`（同一引擎，见 `cimcube-utils` 的 `references/MapSplit.md`）：`MapSplitWidget.getMapSplit()` 返回其实例，也可以按该技能直接 `new U.MapSplit(map, map.layerManager, opts).begin()`——**二者互斥**（同一时刻只允许一个分屏工具在 `begin`）。切换对比图层键为 `viewPortLayersVisible`（运行时键名；个别文档写作 `layersVisible` 非运行时键，以运行时为准）。

## 4. 样式

`MapSplit.css`：`.cimcube-mapsplit-container`、`.cimcube-mapsplit-slider(-horizontal/-vertical)`、`.cimcube-mapsplit-switch`。覆盖示例：`.cimcube-mapsplit-slider-horizontal { background: #e6a23c; }`。
