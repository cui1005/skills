# 飞行控制 · 微调 · 相机环绕自转

入口 `map.camera`。

## 飞行控制与微调

```js
map.camera.cancelFly();          // 取消当前飞行动画（等同内核 camera.cancelFlight）
map.camera.zoomIn();             // 沿视线前进一段（amount 默认 1000，单位米）
map.camera.zoomIn(500);
map.camera.zoomOut();            // 沿视线后退
map.camera.zoomOut(2000);
```

## 相机环绕自转 `startRotate` / `stopRotate`

```
camera.startRotate(center?, duration=60, clockWise=true, loop=true)
  center      // 旋转中心（Cartographic；缺省=围绕地球中心自转）
  duration    // 绕一周用时（秒），默认 60
  clockWise   // true=顺时针，false=逆时针，默认 true
  loop        // 是否循环，默认 true
```

```js
map.camera.startRotate();                 // 地球自转演示：环绕球心旋转，60s/圈
map.camera.startRotate({ longitude: 2.03, latitude: 0.7 }, 30, true, true); // 围绕某点转
map.camera.stopRotate();
```

> ⚠️ 运行时把 `center` 直接交给 `Cartographic.toCartesian`，按**弧度**解释（APIDOC 示例写 `{longitude:120, latitude:26}` 数值会因按弧度解析而偏位；请先转弧度或用 `Cartographic.fromDegrees(116.4, 39.9)` 之类构造）。不确定就省略 center 做全球自转。
