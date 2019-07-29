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

![image-20190727152516840](http://ww4.sinaimg.cn/large/006tNc79gy1g5efeqa26fj30yy0cqgm7.jpg)

### Create an Ascending Index on a Single Index 创建一个升序单索引

​	假设集合**records**中的文档有如下格式：

```js
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```

​	使用以下语句在字段**score**创建升序索引：

```js
db.recordes.createIndex( { score: 1 } )
```

​	索引声明的字段值表述了该字段的索引类型。比如，1表示一个以升序排序的索引；-1则是降序。除此之外还有[其它索引类型](https://docs.mongodb.com/manual/indexes/#index-types)。

​	这个已创建的索引将会支持在字段**score**上的查询，比如：

```js
db.records.find( { score: 2 } )
db.records.find( { score: { $gt: 10 } } )
```

### Create an Index on an Embedded Field 在嵌套字段创建索引

​	你可以在嵌套文档中创建索引，就像你在高层文档索引一样。嵌套字段索引与嵌套文档索引不同，嵌入文档中的索引包括索引中嵌入文档的最大索引大小的完整内容。相反，嵌套字段索引允许使用点符号法来反射到嵌套的文档里。

​	假设有**records**集合，集合中的文档有以下格式：

```js
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```

​	以下操作会在 **location.state** 创建一个索引：

```js
db.records.createIndex( { "location.state": 1 } )
```

​	已创建的索引将会支持查询 **location.state** 字段，例如：

```js
db.records.find( { "location.state": "CA" } )
db.records.find( { "location.city": "Albany", "location.state": "NY" } )
```

### Create an Index on Embedded Document 在嵌套文档中创建索引

​	可以将整个嵌套文档作为一个索引。

​	假设有**records**集合，其中的文档有如下结构:

```js
{
  "_id": ObjectId("570c04a4ad233577f97dc459"),
  "score": 1034,
  "location": { state: "NY", city: "New York" }
}
```

​	**location**字段是一个嵌套文档，包含嵌套字段 **city** 和 **state**。以下命令在**location**上创建一个索引：

```js
db.records.createIndex( { location: 1 } )
```

​	然后可以基于索引查询 **location** 字段：

```js
db.records.find( { location: { city: "New York", state: "NY" } } )
```

### Additional Considerations

​	如果集合有很多数据，并且应用需要在创建索引的同时可以访问这些数据，考虑在背景建立做引，参考 [Background Construction](https://docs.mongodb.com/manual/core/index-creation/#index-creation-background).

​	另外还有其他的关于建立索引的信息：[Index Build Operations on a Populated Collection](https://docs.mongodb.com/manual/core/index-creation/#index-operations), 包括 [Build Indexes on Replica Sets and Sharded Clusters](https://docs.mongodb.com/manual/core/index-creation/#index-operations-replicated-build).

## Compound Indexes 复合索引

​	复合索引是一个引用集合中文档的多个字段的单索引结构（复合索引最多32个字段）。下图显示一个在两个字段上的复合索引：

![image-20190727160604084](http://ww1.sinaimg.cn/large/006tNc79gy1g5egl4s1fgj30xg0cawf9.jpg)

### Create a Compound Index 创建一个复合索引

​	使用一下原型组成的操作可以创建一个复合索引：

```js
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )
```

​	索引声明的字段值表述了该字段的字段类型。比如，值为1表示升序索引，值为-1表示降序索引。

​	假设有**products**集合，文档有如下结构：

```js
{
 "_id": ObjectId(...),
 "item": "Banana",
 "category": ["food", "produce", "grocery"],
 "location": "4th Street Store",
 "stock": 4,
 "type": "cases"
}
```

​	一下操作可以在字段 **item** 和 **stock** 创建一个升序的索引：

```js
db.products.createIndex( { "item": 1, "stock": 1 } )
```

​	在复合索引中字段的罗列顺序很重要。索引会包含对文档的引用，先以**item**字段值进行排序，对于每个**item**字段值再对**stock**进行排序。

​	为了支持匹配所有索引字段的查询，复合索引支持匹配索引字段前缀的查询。也就是说索引支持对于**item**字段查询，同样也支持**item**和**stock**同时查询：

```js
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana", stock: { $gt: 5 } } )
```

### Sort Order 排序顺序

​	索引存储字段值的引用，包括升序或降序。对于单字段索引，键的排序无关紧要，因为MongoDB会以任意方向遍历索引。然而，对于复合索引，排序方向会决定索引是否支持排序操作。

​	假设有集合events，其中文档有字段**username**和**date**。应用可以提出查询，该查询返回先按**username**的值升序然后按**date**的值降序的结果：

```js
db.events.find().sort( { username: 1, date: -1 } )
```

​	或者排序方向相反：

```js
db.events.find().sort( { username: -1, date: 1 } )
```

​	以下创建索引的方式就可以支持以上的两种排序操作：

```js
db.events.createIndex( { "username": 1, "date": -1 } )
```

​	然而，这样建立索引**不能**支持先升序**username**然后再升序**date**：

```js
db.events.find().sort( { username: 1, date: 1 } )
```

### Prefixes 前缀

​	索引前缀是索引字段子集的开端。比如，假设有以下复合索引：

```js
 { "item": 1, "location": 1, "stock": 1 }
```

​	这个索引就有以下索引前缀：

> - { item: 1}
> - { item: 1, location: 1 }

​	对于一个复合索引，MongoDB可以使用索引来支持对索引前缀的查询。正因如此，MongoDB可以使用索引来查询以下的字段：

> - **item**字段
> - **item**字段和**location**字段
> - **item**字段、**location**字段和**stock**字段

​	MongoDB还可以用索引来对**item**和**stock**字段进行查询，因为**item**字段对应了一个前缀。然而，这个索引的效率不如针对**item**和**stock**设置单字段索引高。

​	然而，MongoDB不可以不通过**item**字段而使用索引来查询以下字段，这些字段没有一个对应上索引前缀：

> - **loaction**
> - **stock**
> - **location**和**stock**

​	如果你有一个集合，既有复合索引，并且还有一个基于复合索引中某前缀的单字段索引(例如：{ a: 1 , b: 1 }, { a: 1 } )。如果这两个索引既不稀疏也不唯一，那么你可以删除那个基于前缀的单字段索引。MongoDB将会在所有情况下使用复合索引，这样它就会使用前缀索引。

### Index Intersection 索引交集

[Index Intersection and Compound Indexes](https://docs.mongodb.com/manual/core/index-intersection/#index-intersection-compound-indexes)

## Multikey Indexes 多键索引

