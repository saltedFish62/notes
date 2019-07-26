[TOC]

# Indexes 索引

## Introduce

​	在MongoDB里，索引使得查询的执行更有效率。如果没有索引，MongoDB就要扫描集合，即扫描集合中的每一个文档，来选择那些与查询语句匹配的文档。如果查询使用了一个合适的索引，MongoDB可以使用索引来限制那些必须检查的文档数。

索引是一个特殊的数据结构，它存储便于遍历的集合数据集的一小部分。索引存储一个特定或者字段集合的值，它们会根据字段的值排序。索引条目的顺序化支持高效的相等匹配和基于范围查询的操作。另外，MongoDB可以使用索引的顺序返回一个排好序的结果。

下图展示了一个使用索引来选择和排序文档的查询：

![image-20190726153657149](http://ww3.sinaimg.cn/large/006tNc79gy1g5da4l52bfj312s0jymz3.jpg)

​	基本的，在MongoDB里的索引与其他数据库系统的索引是相似的。MongoDB在集合层次定义索引并且支持在集合中的文档的任意字段或子字段设置索引。

### Default _id Index 默认的\_id索引

MongoDB会在创建一个集合时，在字段\_id创建一个值唯一的索引。\_id索引还会组织客户端插入两个\_id字段值相等的文档。你不能删除\_id字段上的索引。

### Index Types 索引类型

MongoDB提供多种索引类型来支持特定的数据类型和查询。

#### Single Field 单字段

​	除了MongoDB定义的\_id字段以外，MongoDB还支持创建用户定义的降序或升序的单字段索引。

比如：

![image-20190726154858445](http://ww2.sinaimg.cn/large/006tNc79gy1g5dah1fs5aj31020dut9c.jpg)

对于单字段索引和排序操作，索引键的排序顺序不会有影响，因为MongoDB可以在任意方向上遍历索引。

#### Compound Index 复合索引

​	MongoDB也支持用户定义多字段索引，即复合索引。

​	复合索引中字段的罗列顺序是有意义的。比如说，如果复合索引为: 

```js
{ userid: 1, score: -1 }
```

​	那么索引将会先根据userid排序，然后对于每个userid的值对score进行排序。

![image-20190726155925677](http://ww1.sinaimg.cn/large/006tNc79gy1g5darxd6k9j312i0e8t9o.jpg)

​	对于复合索引和排序操作，索引键的排序顺序可以决定这个索引是否可以支持排序操作。

#### Multikey Index 多键索引

MongoDB使用多键索引来索引存储在数组中的内容。如果你索引一个值为数组的字段，MongoDB会为数组中的每个元素建立分离的索引条目。这些多键索引允许查询通过匹配数组的单个元素或多个元素来选择包含数组的文档。MongoDB会自动决定是否创建多键索引如果索引的字段的值为数组，不去要特地声明多键索引。

![image-20190726164558786](http://ww4.sinaimg.cn/large/006tNc79gy1g5dc4cxsx4j30yc0j4my2.jpg)

#### Geospatial Index 地理空间索引

为了支持高效的地理坐标数据查询，MongoDB提供两个特殊的索引：`2d indexes`和`2dsphere indexes`。

2d Indexes 返回结果时使用平面几何；2dsphere indexes 返回结果时使用球面几何。

#### Text Indexes 文本索引

MongoDB提供一个 **text** 索引类型，该类型支持查找集合中字符串内容的搜索。这些文本索引不会存储语言特定的停止词并且在及合理只存储根词汇。

#### Hashed Indexes 哈希索引

为了支持基于哈希的切片，MongoDB提供了一个哈希索引类型，它索引的是字段值的哈希值。这些索引在值域中有更随机的分布，但值支持等值匹配并且不支持基于范围的索引。

### Index Properties 索引属性

#### Unique Indexes 唯一索引

索引的唯一性使MongoDB拒绝索引字段出现重复值。出了唯一性的约束，唯一索引的功能和其他MongoDB的索引一样。

#### Partial Indexes 部分索引

部分索引只索引集合中满足特定过滤表达式的文档。由于只索引集合中的文档子集，部分索引对存储需求更低，并且减少了创建索引和维护的性能支出。

#### Sparse Indexes 稀疏索引

索引的稀疏属性确保索引只包含有该索引字段的文档。这种索引跳过了不含该索引字段的文档。

#### TTL Indexes 生存时间索引

MongoDB可以在过了一段特定的时间后，自动删除集合中的文档。

## Single Field Indexes 单字段索引

​	MongoDB完整地支持在集合中的文档的任意字段设置索引。默认的，所有集合都在\_id字段上设置索引，并且应用和用户可以添加另外的索引来支持重要的查询和操作。
