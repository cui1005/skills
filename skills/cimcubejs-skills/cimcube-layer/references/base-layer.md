# BaseLayer — 所有图层的公共契约（父类）

入口 `map.layerManager`。

## 构造选项

| 键 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `id` | String | — | 图层 id（建议显式指定，addLayer 幂等依赖它） |
| `name` | String | — | 图层名称 |
| `type` | String | **必填** | `Defines.LayerType.*` |
| `url` | String | — | 数据地址 |
| `show` | Boolean | `true` | **显隐（唯一显隐属性）** |
| `alpha` | Number | `1.0` | **透明度** 0.0~1.0 |
| `enableMouseMove` | Boolean | `false` | 是否派发鼠标移动事件 |
| `canSelected` | Boolean | `false` | 是否可选中 |
| `canHighlight` | Boolean | `false` | 选中是否高亮 |
| `selectedColor` | String | `"#32D3A6"` | 点选颜色 |
| `glowColor` | String | `"#FF0000"` | 发光颜色 |
| `glowTimeInterval` | Number | `3000` | 闪烁间隔（ms） |
| `pickable` | Boolean | `true` | 是否可拾取 |
| `hiddenInLayerTree` | Boolean | `false` | `true` 时不进入图层树枚举（`getAllLayers`/`getLayersByType` 看不到，但 `getLayer` 仍可取到） |

## 成员（读写）

`id`(ro) · `name` · `type`(ro) · `url`(ro) · **`show`** · **`alpha`** · `enableMouseMove` · `canSelected` · `canHighlight` · `selectedColor` · `glowColor` · `glowTimeInterval` · `pickable` · `cacheable` · `hiddenInLayerTree`(ro) · `matrix`(ro) · `rotationPointOffset` · `isPipeline` · `ready`(ro) · `dragable`(ro)

## 方法

| 方法 | 说明 |
|---|---|
| `addTo(map)` | 挂到图层管理器（等价 `map.layerManager.addLayer(this)`） |
| `removeFrom()` | 从图层管理器移除 |
| `getParentId()` | 父节点 id |
| `removeDragHandle()` | 移除场景中的操作句柄 |
| `getBoundingSphere()` / `flyTo()` | 基类为空实现，由子类覆写 |

## 事件订阅（EventDispatcher）

```js
const D = Glodon.CIMCube.Global.Defines;
const off = layer.on(D.LayerEventType.LOADED, (e) => console.log("loaded", e));
layer.off(D.LayerEventType.LOADED, off);   // 或 layer.removeEventListener(type, fn)
layer.fire({ type: D.LayerEventType.LOADED }); // 手动派发（等价 dispatchEvent）
```

`on(type, listener, thisArg)` 返回取消函数；`off(type, listener?)` 省略 listener 时移除该类型全部监听。

> 事件取值表与各图层可用事件见 `references/events.md`。
