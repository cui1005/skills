# Sunlight — 日照模拟（可运行示例）

> 前提：`map` 已创建；`const A = Glodon.CIMCube.Analysis`。
> 随时间推进模拟日照光影。**options 字段（类页）：**

| 字段 | 类型 | 说明 |
|---|---|---|
| `startTime` | `Date` | 日照起始时间 |
| `endTime` | `Date` | 日照结束时间 |
| `multiplier` | `Number` | 时间倍数（默认 1.0），调时间快慢 |

**主要方法：** `doAnalysis()`、`pause()`、`stop()`、`setStartTime(date)`、`setEndTime(date)`、`setCurrentTime(date)`、`setMultiplier(value)`、`dispose()`。

```js
const sun = new A.Sunlight(map, {
  startTime:  new Date(2026, 5, 21, 6, 0),   // 日照起始时间（Date）
  endTime:    new Date(2026, 5, 21, 19, 0),  // 日照结束时间（Date）
  multiplier: 1.0,                           // 时间倍数，调快慢（默认 1.0）
});
sun.doAnalysis();                          // 开始日照模拟
sun.setMultiplier(120);                    // 调快时间倍率，快速预览全时段
sun.pause();                               // 暂停
sun.setCurrentTime(new Date(2026, 5, 21, 12, 0)); // 直接跳到正午观察光影
sun.stop();                                // 停止
sun.dispose();                             // 释放资源
```
