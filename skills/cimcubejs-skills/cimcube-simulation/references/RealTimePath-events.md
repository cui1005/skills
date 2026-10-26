# RealTimePath 方法与事件

> 回主路由：`../SKILL.md`；构造 options 与数据流见 `RealTimePath.md`；数据/帧协议/演示生成器见 `RealTimePath-data.md`。

## 2.4 方法与事件

```js
rtp.on("featureRendered", (p) => {        // 智能体模型首次可渲染（on = addEventListener，返回注销函数）
  console.log("渲染", p.featureID);
});
rtp.on("featureDisposed", (p) => console.log("销毁", p.featureID, p.model)); // 模型实例（RealTimePathEntity）
rtp.on("featureMoved", (p) => console.log("移动", p.featureID, p.cartographicDegrees));      // {lon,lat,height}
rtp.on("propertiesChanged", (p) => console.log("属性变化", p.featureID, p.properties));
const one = rtp.getFeatureModel("featureId0_0");   // RealTimePathEntity
one.on("featureMoved", (p) => { /* p: {featureID, cartographicDegrees} —— 单实体通道 */ });
one.on("propertiesChanged", (p) => { /* p: {featureID, properties} */ });
rtp.getFeaturesProperties("featureId0_0"); // { cartographicDegrees, animationName, properties: {...} }
rtp.getAllFeaturesModel();                  // 当前全部实体 map {featureID: RealTimePathEntity}
rtp.showPathByPolylines("featureId0_0", {   // 显示整条漫游 debug 线（动态线，不影响模型）
  lineType: 3,     // 默认 3：0 实线 | 1 发光线 | 2 虚线 | 3 动态线
  duration: 5,     // 持续(秒)：文档默认 5；运行时默认 1
  lineWidth: 20,   // 默认 20
  lineColor: "#00C6FF",  // 默认 #00C6FF
  image: "",       // 动态线贴图（默认 Resource/animation/line/dynamic.png）
  maskLength: 25,  // 默认 25
});
rtp.showPathByPolylines();                  // 省略 featureID = 全部智能体
rtp.clearFeaturesModel();                   // 清全部（逐个 featureDisposed）；传 featureID 清单个
rtp.dispose();                              // 停 tick、清全部、销毁实例集合与 Primitive
```

> **事件双通道**：引擎把每个实体事件**转发**一份到 `RealTimePath` 上（`featureMoved/propertiesChanged` 转发时略去个别字段，见上）；实体自身也是 `EventDispatcher`，可用 `entity.on(...)` 精确监听单智能体。
> `EventDispatcher` 语义：`on == addEventListener`，返回可调用的注销函数；另有 `off/removeEventListener/removeEventListener(types)/fire/dispatchEvent/hasEventListener`（见 cimcube-global 技能）。
