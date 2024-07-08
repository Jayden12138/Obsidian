
> https://geojson.io/ ：主要是用来预览geojson，可以进行微调，操作方式挺有趣的
> https://www.naturalearthdata.com/ ：数据来源（最新更新日期是2018，但命名存在一定的问题，官方说是更国际认可的命名）
> QGIS：用来看.shp/.geojson/.json文件


update:
0708: 甲方提供了一版精确度较为高的世界地图，与原系统中使用的中国地图（带省份）边界几乎重合

在QGIS中，通过excel 链接，给世界地图添加了中文名称字段
使用jq 合并两份数据
- 修正省份名称
- 世界地图中删除中国
- 中国国界线命名（点选时需要过滤）


---

主要命令：

```shell

# merge
jq -s '.[0].features + .[1].features' sourve1.geojson source2.geojson | jq '{type: "FeatureCollection", features: .}' > target.geojso

#压缩
jq -c '.' target.geojson > output.json

```




---

## TODO

- [x] jq 命令 ✅ 2024-07-03
- [x] 线转面后 重命名（暂无方式简化） ✅ 2024-07-03

## 前言

>  原因：原系统geojson数据，国内省份边界数据精度远大于国外国家边界数据，直接覆盖使用会造成边界冲突


准备数据：
ne_10m_admin_1_states_provinces.zip // 1-10m 全部省份/州 边界
ne_110m_admin_0_countries_lakes.zip // 1-110m 全球国家边界

## 大致流程

### filter

ne_10m_admin_1_states_provinces_lines.zip 
=> 国内各省份之间的边界数据（1:10m line）

ne_110m_admin_0_countries_lakes.zip 
=> 中国边界数据（1:110m polygon)
=> polygon to line

### merge

> 合并前可以对 国内各省份之间的边界数据 进行 简化操作，
> 好处：
> 	1. 降低精度，与国外国家边界在观感上减少对比度
> 	2. 数据量大大减少


通过 jq 合并上述两组数据

```shell

jq -s '.[0].features + .[1].features' sourve1.geojson source2.geojson | jq '{type: "FeatureCollection", features: .}' > target.geojson

```

### line to polygon

> 因为源数据的精度不一致，大概率存在空隙，需要在QGIS中手动补齐后，再进行转换操作）

// TODO：有点费事，暂无更好的方式
这里转换成面后，因为 省份间的边界数据中，只有线左右两侧的省份名称，不能很好标记面的名称，需要对面进行命名（add attribute）

### last

导出geojson

## 番外

### jq

https://jqlang.github.io/jq/

```shell

# MacOS
# https://jqlang.github.io/jq/download/

# install jq
brew install jq

# 将要进行合并操作的两个geojson文件：sourve1.geojson source2.geojson

jq -s '.[0].features + .[1].features' sourve1.geojson source2.geojson | jq '{type: "FeatureCollection", features: .}' > target.geojson


```

命令详解：
1. **`jq -s '.[0].features + .[1].features' sourve1.geojson source2.geojson`**：
    - `jq` 是一个处理 JSON 的命令行工具。
    - `-s` 参数表示将输入的多个 JSON 文件作为一个数组来处理。
    - `'.[0].features + .[1].features'` 是一个 jq 表达式，表示获取输入数组中的第一个和第二个元素的 `features` 属性，并将这两个 `features` 数组相加。
    - `source1.geojson` 和 `source2.geojson` 是两个输入的 GeoJSON 文件。

    这部分命令的作用是从两个 GeoJSON 文件中提取 `features` 数组，并将它们合并成一个数组。

2. **`| jq '{type: "FeatureCollection", features: .}'`**：
    - 这部分命令通过管道(`|`)将前一步的输出传递给另一个 `jq` 命令。
    - `jq '{type: "FeatureCollection", features: .}'` 表示将合并后的 `features` 数组包装成一个新的 JSON 对象，其中 `type` 属性被设置为 `"FeatureCollection"`，`features` 属性的值是前一步得到的合并后的数组。

    这部分命令的作用是将合并后的 `features` 数组构造成一个新的 GeoJSON 对象。

3. **`> target.geojson`**：
    - `>` 是一个重定向操作符，表示将前一步的输出写入到 `target.geojson` 文件中。

    这部分命令的作用是将最终的 GeoJSON 对象保存到一个新的文件中。



```shell

# 压缩json文件

jq -c '.' target.geojson > output.json



```