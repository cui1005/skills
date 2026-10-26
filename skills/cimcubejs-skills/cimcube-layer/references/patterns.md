# 常见模式（可套用配方）

入口 `map.layerManager`。
**① 加载底图 + 地形 + 模型（最典型初始化）**

```js
const D = Glodon.CIMCube.Global.Defines;
const LM = map.layerManager;
LM.addLayer({ type: D.LayerType.IMAGERY, id: "img", name: "影像",
  url: "http://.../{z}/{x}/{y}.png", imageryType: "xyz" });
LM.addLayer({ type: D.LayerType.TERRAIN, id: "ter", name: "地形",
  url: "http://.../terrain" });
const bim = LM.addLayer({ type: D.LayerType.TILES3D, id: "bim", name: "BIM",
  url: "./data/tileset.json" });
bim.on(D.LayerEventType.LOADED, () => LM.flyToLayer("bim"));
```

**② 图层树 / 显隐 / 排序**

```js
LM.showLayer("bim", false);              // 隐藏
LM.showLayers(["bim", "img"], true);     // 批量显示
LM.renameLayer("bim", "主体模型");
LM.moveLayer("bim", "g1", 0);            // 移到分组 g1 的第 0 位
const layers = LM.getLayersByType(D.LayerType.TILES3D);
```

**③ 分组批量管理**

```js
const g = LM.addLayer({ type: D.LayerType.GROUP, id: "g1", name: "一期工程" });
LM.addLayer({ type: D.LayerType.TILES3D, id: "b1", name: "A栋", url: "a.json" }, "g1");
LM.addLayer({ type: D.LayerType.TILES3D, id: "b2", name: "B栋", url: "b.json" }, "g1");
LM.getLayer("g1").show = false;          // 整组隐藏
```

**④ 整体变换（方案对比）**

```js
const tg = new Glodon.CIMCube.Layer.TransformableGroupLayer({ children: ["b1", "b2"] }, map);
LM.addLayer(tg);
tg.offset = [50, 0, 0];
tg.rotation = [0, 0, Math.PI / 6];
tg.show = false;
```

**⑤ 点选实体取属性**

```js
geoLayer.on(D.LayerEventType.ENTITY_CLICK, (e) => {
  const { pickId, featureInfo } = e.arguments;
  console.log("点中要素", pickId, featureInfo);
});
```

**⑥ 移除与释放**

```js
LM.removeLayer("bim");                   // 仅移除
LM.removeLayer("bim", true);             // 连资源销毁
LM.removeLayersByType(D.LayerType.TILES3D);
LM.removeAllLayers();
```
