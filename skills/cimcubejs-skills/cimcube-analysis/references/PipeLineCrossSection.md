# PipeLineCrossSection — 管线横断面（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）；场景已加载地下管线数据。
> 指定一条切割线段与竖向深度，对范围内的地下管线做**横断面剖切**：生成断面图（base64 图片）并返回每条管道的空间属性，供绘制剖面图或净距校核。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `line` | `Array` | — | 剖切线段两点：`[{lon,lat,height}, {lon,lat,height}]`（或二维数组） |
| `depth` | `Number` | `50` | 向下剖切深度（米，`>0`） |
| `length` | `Number` | `2` | 纵向（沿线）取数范围（米） |
| `volumeColor` | `String`/`Color` | `#00ffff` | 剖切体立面颜色 |
| `volumeAlpha` | `Number` | `0.3` | 剖切体透明度 |

方法：`setLine(line)` / `setDepth(depth)` / `setLength(length)` / `setVolumeColor(color)` / `setVolumeAlpha(alpha)`；`doAnalysis(callback)` 执行并回调结果；`clear()` 清空结果；`dispose()` 销毁。

回调参数 `event` 字段：`event.dataBase64` 断面图片（可直接赋给 `Image`）；`event.distances` 各管线水平间距；`event.height`/`event.width` 断面图高/宽；`event.widthInMeter`；`event.offset` 偏移（默认 `[100, 100]`）；`event.objInfo` 管线对象数组——每项含 `alias / layerId / shape("rectangle"|"circle") / width / height / radius / depth / center / color / box / featureInfo(userId) / pickedObject` 等。

```js
const cps = new A.PipeLineCrossSection(map, {
  line: [
    { lon: 120.0, lat: 38.0, height: 20 },
    { lon: 120.001, lat: 38.0, height: 20 },
  ],
  depth: 50,
  length: 2,
  volumeColor: "#00ffff",
  volumeAlpha: 0.3,
});
cps.doAnalysis((event) => {
  const img = new Image();
  img.src = event.dataBase64;            // 断面渲染图
  document.body.appendChild(img);
  console.log(event.objInfo);            // 断面内每条管道的对象信息
  console.log(event.distances);          // 管线间距
});
// 修改剖切位置后重做：
// cps.setLine([...]); cps.doAnalysis((e) => {...});
// cps.clear();
// cps.dispose();
```
