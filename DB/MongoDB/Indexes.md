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

​	为了索引一个值为数组的字段，MongoDB会给数组中的每个元素都建立一个索引。这些多键索引可以针对数组字段值进行高效索引。可以在包含标量值(比如字符串和数字)和嵌套文档的数组建立多键索引。

![image-20190729200058337](http://ww1.sinaimg.cn/large/006tNc79gy1g5gym7vl0uj30y80j2wfe.jpg)

### Create Multikey Index 创建多键索引

​	使用 `db.collection.createIndex()` 来创建多键索引：

```js
db.coll.createIndex( { <field>: < 1 or -1> } )
```

​	如果任意被索引的字段是一个数组，MongoDB会自动地创建多键索引；你不需要特别地去声明多键索引。

### Multikey Index Bounds 多键索引边界

​	索引扫描的边界定义了一个查询中索引搜索的范围。当一个已存在的索引有多个谓词时，MongoDB会尝试用取交集或者组合的手段组合这些谓词的边界，以此缩小扫描的边界。

#### Intersect Bounds for Multikey Index 多键索引边界取交集

​	边界的交集指多个边界的逻辑与。例如：有两个边界 [ [ 3, Infinity ] ] 和 [ [ -Infinity, 6 ] ]，那么这两个边界的交集就是 [ [ 3, 6 ] ]。

​	给定一个索引的数组字段，考虑有一个在数组上声明多前缀并且可以使用多键索引的查询。如果一个[`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#op._S_elemMatch)联合这些谓词，MongoDB可以对多键索引的边界取交集。

​	比如，一个survey集合有包含字段 **item** 和字段 **ratings** 的文档：

```js
{ _id: 1, item: "ABC", ratings: [ 2, 9 ] }
{ _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
```

​	给 **ratings** 数组创建一个多键索引：

```js
db.survey.createIndex( { ratings: 1 } )
```

​	接下来的查询使用 `$elemMatch` 来获取数组中包含至少一个元素满足两个条件的文档：

```js
db.survey.find( { ratings: { $elemMatch: { $gte: 3, $lte: 6 } } } )
```

​	接下来讨论每一个谓词：

- 大于等于3的谓词(即 **$gte: 3**)的边界是: [ [3, Infinity ] ];
- 小于等于6的谓词(即 **$lte: 6**)的边界是：[ [ -Infinity, 6 ] ]

​    由于查询使用 `$elemMatch` 来联结这些谓词，MongoDB可以给这些边界取交集：

```js
ratings: [ [ 3, 6 ] ]
```

​	如果查询没有联结这些条件，MongoDB不能取索引边界的交集，查询如下：

```js
db.survey.find( { ratings: { $gte: 3, $lte: 6 } } )
```

​	查询会搜索 **ratings** 数组，找满足至少一个元素大于等于3并且至少一个元素小于等于6的数组。由于单一个元素不需要同时满足两个边界，MongoDB不会取边界的交集而是使用[ [ 3, Infinity ] ]或者[ [ -Infinity, 6 ] ]。MongoDB不保证使用这两个边界的哪个。

#### Compound Bounds for Multikey Index 多键索引的复合边界

​	复合边界是指使用复合索引的多键的边界。比如，给定一个字段 **a** 有边界 [ [ 3, Infinity ] ] 并且字段 **b** 有边界 [ [ -Infinity, 6 ] ] 的复合索引 **{ a: 1, b: 1 }**，复合边界会使用两个边界：

```js
 { a: [ [ 3, Infinity ] ], b: [ [ -Infinity, 6 ] ] }
```

​	如果MongoDB不可以复合两个边界，则它总是使用首个字段的边界来约束索引扫描的范围，在这个例子中，边界就是 **a: [ [ 3, Infinity ] ]**。

##### Compound Index on an Array Field 数组字段上的复合索引

考虑一个复合多键索引，即其中一个索引字段为数组的复合索引。比如，一个 **sorvey** 集合，其中的文档有字段 **item** 和字段 **ratings**：

```js
{ _id: 1, item: "ABC", ratings: [ 2, 9 ] }
{ _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
```

​	在字段 **item** 和 **ratings** 上建立复合索引：

```js
db.survey.createIndex( { item: 1, ratings: 1 } )
```

​	以下查询声明了对两个键的条件：

```js
db.survey.find( { item: "XYZ", ratings: { $gte: 3 } } )
```

​	单独考虑每个谓词：

- **item** 的边界： 谓词"XYZ"为 [ [  "XYZ", "XYZ" ] ];
- **ratings** 的边界：谓词 { $gte: 3 } 为 [ [ 3, Infinity ] ]

​    MongoDB可以通过联结复合两个边界：

```js
{ item: [ [ "XYZ", "XYZ" ] ], ratings: [ [ 3, Infinity ] ] }
```

##### Compound Index on Fields from an Array of Embedded Documents 在嵌套文档数组中的字段上的复合索引

​	如果一个数组包含了嵌套文档，用[点字段名](https://docs.mongodb.com/manual/core/document/#document-dot-notation)的方式来给数组中的嵌套文档的字段建立索引。

```js
ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]
```

​	**score**字段的点字段名为 **"ratings.score"**。

###### Compound Bounds of Non-array Field and Field from an Array

​	假设有集合 **survey2**，其中文档有字段 **item**，和数组字段 **ratings**。

```js
{
  _id: 1,
  item: "ABC",
  ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]
}
{
  _id: 2,
  item: "XYZ",
  ratings: [ { score: 5, by: "anon" }, { score: 7, by: "wv" } ]
}
```

​	在非数组字段 **item** 和 **ratings** 里的两个字段 **ratings.score** 和 **ratings.by** 创建复合索引。

```js
db.survey2.createIndex( { "item": 1, "ratings.score": 1, "ratings.by": 1 } )
```

​	对这三个字段查询：

```js
db.servey2.find( { item: "XYZ", "ratings.score": { $lte: 5 }, "ratings.by": "anon" } )
```

​	对每个谓词来说：

- **item** 的边界：谓词 "XYZ"是 [ [ "XYZ", "XYZ" ] ];
- **score** 的边界：谓词 { $lte: 5 } 是 [ [ -Infinity, 5 ] ];
- **by** 的边界：谓词 "anon" 是 [ "anon", "anon" ].

​    MongoDB可以复合 **item** 键的，和 **ratings.score** 或者 **ratings.by** 两者其中之一的边界，取决于查询的谓词和索引键值。MongoDB不保证后两者谁会与 **item** 键复合。比如，**item**键可能与 **ratings.score** 复合，也可能与 **ratings.score** 复合。

​	然而，查询如果要将 **"ratings.score"** 和 **"ratings.by"** 的边界复合，则必须使用 `$elemMatch`。

###### Compound Bounds of Index Fields from an Array 复合数组中的索引字段的边界

​	要把相同数组的索引键的边界复合起来：

- 索引键必须共用同一字段路径但不包括字段名，并且
- 查询必须在该路径上使用 `$elemMatch` 在字段上指定谓词。

​    对于在嵌套文档中的一个字段，例如 **"a.b.c.d"** 的点字段名是 **d** 的字段路径。要复合相同数组中的索引键的边界，`$elemMatch`必须在路径上但不能有字段名本身，即 **"a.b.c"**。

### Unique Multikey Index 唯一多键索引

​	对于唯一索引，唯一性约束适用于集合中的多个文档而不是单独一个文档。

​	因为唯一性约束适用单个文档，那么对于唯一多键索引，如果一个文档的索引键值不是从别的文档复制过来的，该文档就可能会有导致重复的索引键值的数组元素。

### Limitations 限制

#### Compound Multikey Indexes 复合多键索引

​	对于复合多键索引，每个索引的文档可以有最多一个值为数组的索引字段。也就是说：

- 如果有多于一个值为数组的字段值，则不能创建复合多键索引。比如以下文档：

  ```js
  { _id: 1, a: [1, 2], b: [1, 2] }
  ```

  则不能在集合上创建复合多键索引 `{ a: 1, b: 1 }`，因为**a**和**b**字段的值都是数组。

- 或者，如果复合多键索引已存在，你不能插入一个违反该限制的文档。

  比如以下集合中有这样的文档：

  ```js
  { _id: 1, a: [1, 2], b: 1, category: "A array" }
  { _id: 2, a: 1, b: [1, 2], category: "B array" }
  ```

  一个复合多键索引 `{ a: 1, b: 2 }` 是每个文档都有的，只有一个被复合多键索引索引的字段是一个数组，也就是没有文档的 **a** 和 **b** 的值都是数组。然而，创建复合多键索引之后，如果你尝试插入一个 **a** 和 **b** 字段的值都是数组的文档时，MongoDB会把这次插入失效处理。

如果一个字段是一个文档数组，你可以索引签到的字段来创建复合索引。比如，有包含以下文档的集合：

```js
{ _id: 1, a: [ { x: 5, z: [ 1, 2 ] }, { z: [ 1, 2 ] } ] }
{ _id: 2, a: [ { x: 5 }, { z: 4 } ] }
```

​	你可以创建一个复合索引： `{ "a.x": 1, "a.z": 1 }`。最多只有一个值为数组的索引字段的限制仍然适用。

### Sorting 排序

​	当对一个使用多键索引的数组排序时，查询计划中会包含一个阻塞的 **SORT** 阶段。新的排序行为可能会对性能产生负面影响。

​	在阻塞的 **SORT**，所有的输入必须经过排序步骤才可以输出。在一个非阻塞或索引排序中，排序步骤扫描索引来产生按要求排序的结果。

#### Shard Keys 分片的键

​	不能将多键索引声明为分片索引键。然而如果分片索引键是一个复合索引的前缀，且除了分片的键之外的一个键的值是数组，就允许复合索引变成一个复合多键索引。复合多键索引会影响性能。

#### Hashed Indexes 哈希索引

​	哈希索引不能作为多键索引。

#### Query on the Array Field as a Whole 以数组字段的全部作为查询

​	当一个查询的过滤声明一个 [整个数组的精确匹配](https://docs.mongodb.com/manual/tutorial/query-arrays/#array-match-exact)，MongoDB可以使用多键索引来查询数组的第一个元素但不能使用多键索引扫描来发现整个数组。相反，使用多键索引来查找数组的第一个元素之后，MongoDB将关联的文档都加载出来，并且将文档中的数组符合查询条件的过滤出来。

​	比如，有 **inventory** 集合包含以下文档：

```js
{ _id: 5, type: "food", item: "aaa", ratings: [ 5, 8, 9 ] }
{ _id: 6, type: "food", item: "bbb", ratings: [ 5, 9 ] }
{ _id: 7, type: "food", item: "ccc", ratings: [ 9, 5, 8 ] }
{ _id: 8, type: "food", item: "ddd", ratings: [ 9, 5 ] }
{ _id: 9, type: "food", item: "eee", ratings: [ 5, 9, 5 ] }
```

​	该集合在字段 **ratings** 上有一个多键索引：

```js
db.inventory.createIndex( { ratings: 1 } )
```

​	以下查询 **ratings** 字段值为 [ 5, 9 ] 的文档：

```js
db.inventory.find( { ratings: [ 5, 9 ] } )
```

​	MongoDB可以用多键索引来找到 **ratings** 数组中包含 **5** 的所有文档。然后，MongoDB加载出来这些文档，并且过滤出 **ratings** 数组等于 **[ 5, 9 ]** 的文档。

### See More Examples

​	[Examples](https://docs.mongodb.com/manual/core/index-multikey/#examples)

## Text Indexes 文本索引

