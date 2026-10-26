# MapLegendWidget 地图图例面板（+ 内部 MapLegend）

## 1. 构造与挂载

`options` **必传**（缺 `sources` 时内部会抛错提示 options 必需）。文档键：

```js
const legend = new W.MapLegendWidget({
  title: "图例",                 // String 可选，默认 "图例"
  sources: [ /* 见 2. 数据源结构 */ ],   // Array 必填：图例数据源
  onclose() {},                 // function 可选，关闭面板时回调
  oncollapse() {},              // function 可选，折叠/展开时回调
}).addTo(map);                  // addTo(map)——注意本类不是 addToMap
// 内部实例挂在 map.mapLegend（只读访问器），面板显隐/数据增删都走 widget 自身 API 即可
legend.show = false;            // show 是 Boolean 成员（读/写皆可）
```

## 2. 数据源结构（键已从运行时渲染逻辑实证）

```js
{
  layerID: "图层ID",                        // getDataSource(id) 也按此 ID 查
  layerName: "显示名",                      // 图例分组标题（可含 fieldDes 说明）
  fieldDes: "地类名称",                     // 可选，样式字段说明文字
  style: {
    geometryType: "POLYGON",                // "POINT" | "POLYLINE" | "POLYGON"
    field: "name",                          // 可选：分类依据的要素属性字段（用于点击联动）
    fillColor: "rgba(86,172,66,.7)",        // 面/点用
    outline: { color: "rgba(255,255,255,.9)", /* 线宽等 */ },  // 描边
    // 线层：color: "rgba(..)"；图标/矢量点：type: "icon"|"vector" + symbol: 图标 url/符号
    items: [                                // 可选：分类（多值）样式数组
      { value: "林地", fillColor: "rgba(86,172,66,.7)", /* 其它渲染键同上 */ },
      { value: "草地", fillColor: "rgba(240,200,60,.7)" }
    ]
  }
}
```

两种形态与点击联动（图例即开关）：

- **均匀样式**（无 `items`）：渲染单色图例块 + `layerName`；点击切换**整层**显隐。
- **分类样式**（`style.items` 有值，每项带 `value` + 渲染键）：折叠条渲染为"每值一行"，点击按 `style.field == item.value` 做**要素级**显隐（内部走 `layerManager` 的属性查询，命中即隐藏/显示）。

## 3. 数据增删与事件

```js
legend.addDataSource({ layerID: "id2", layerName: "b", style: { geometryType: "POLYLINE", color: "rgba(0,140,255,1)" } });
legend.getDataSource("id2");      // 按 图层ID 取；getDataSources() 返回全部
legend.removeDataSource("id2");   // Boolean
legend.removeDataSources();
legend.setDataSources([ ... ]);
legend.dataSourceAdded.addEventListener((ds) => console.log("add", ds));    // EventProperty
legend.dataSourceRemoved.addEventListener((ds) => console.log("remove", ds));
legend.dispose();                 // 移除面板 DOM 并清 map.mapLegend（勿再访问）
```

> 运行时另有默认开启的 `closeBtn` / `miniBtn`（折叠钮）开关与空状态占位图（sources 为空时显示）；这两个键 API_DOC 未列，仅在 API_DOC 已列出的 `onclose/oncollapse` 等之上按需覆盖，勿依赖。
> `MapLegend`（无后缀）是内部实现类，**未导出**，不可 `new`。

## 4. 样式

`MapLegend.css`：`.cimcube-maplegend-container`（右下 310px 宽）> `.cimcube-maplegend-head`（标题/折叠/关闭）+ `.cimcube-maplegend-body`（accordion 分组 `.cimcube-accordion`，每条含折叠箭头、`.cimcube-legend` 色块/图标与分类行）。示例图例建议直接对接 `Utils.MapLegendService`（生成符号 SVG）与图层样式。
覆盖示例：`.cimcube-maplegend-container { bottom: 50px; }`。
