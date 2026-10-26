# 常见模式 · 实用提示

入口 `map.camera`。

## 模式 A：页面初始视角（构造即定位，也可运行后再 setView）

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "./dist/",
  center: { lon: 116.4074, lat: 39.9042, height: 3000, pitch: -Math.PI / 4 },
});
```

## 模式 B：数据加载完成后飞向模型

```js
const layer = map.layerManager.getLayer("model-id"); // 或 addLayer 后持有
map.camera.flyTo({ target: layer, duration: 2, complete: () => map.camera.trackedModel = layer });
```

## 模式 C：截图下载当前画面

```js
// 前提：构造时 params.contextOptions.webgl.preserveDrawingBuffer = true
function downloadView() {
  const s = map.camera.getCaptureImg();
  if (!s) return alert("截图失败：请确认已开启 preserveDrawingBuffer");
  const a = document.createElement("a");
  a.href = s.image; a.download = "snap.png"; a.click();
}
```

## 模式 D：视角书签/分享（存 → 还原）

```js
function saveView() {
  localStorage.setItem("view", map.camera.getCameraInfo(true).cameraCopy);
}
function restoreView() {
  // cameraCopy 形如 map.camera.flyTo({target:{height,lat,lon},orientation:{heading,pitch,roll}})
  eval(localStorage.getItem("view"));
}
```

## 模式 E：摄像机环绕建筑展示

```js
map.camera.flyTo({ target: buildingLayer });                    // 先到建筑上方
map.camera.startRotate(buildingSphereCenter, 40, true, true);   // 绕圈展示，40s/圈
// 展示结束：
map.camera.stopRotate();
```

## 实用提示

- 想"长焦"效果：把 `camera.fov` 调小（如 30）。
- 读取真实姿态一律用 `getCameraInfo()`（弧度）；页面展示需自行转度（乘 `180 / Math.PI`）。
- `changed` 事件高频触发，做 URL 状态同步时注意节流。
