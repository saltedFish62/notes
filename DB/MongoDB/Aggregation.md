[TOC]



---

# Aggregation 聚合

​	聚合操作处理数据记录并且返回计算后的结果。聚合操作可以把多个文档的值组合到一起，并且对组合后的数据执行一系列操作最终返回一个结果。MongoDB提供了三种聚合的方式： Aggregation pipeline, Map-reduce function 和  single purpose aggregation methods。

## Aggregation pipeline 聚合流水线

​	聚合管道是数据聚合框架，基于数据处理流水线概念建模。

例如：

```js
db.orders.aggregate([
   { $match: { status: "A" } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```

**第一阶段：** _$match_ 阶段会过滤出 **status** 字段为"A"的文档然后传递给下一阶段。

**第二阶段：** _$group_ 阶段会根据 **cust_id** 字段来分组，然后计算每一组的 **amount** 字段的总和。

### pipeline 流水线/管道

​	流水线由多个阶段组成。每个阶段会在文档通过该阶段时处理文档。流水线阶段不需要为每一个输入文档创建一个输出文档。流水线的阶段可以在流水线中重复出现多次。阶段的工作可以参考[pipeline stages](https://docs.mongodb.com/manual/reference/operator/aggregation-pipeline/#aggregation-pipeline-operator-reference).

### pipeline Expressions 流水线表达式

​	某些流水线阶段可以将流水线[表达式](https://docs.mongodb.com/manual/meta/aggregation-quick-reference/#aggregation-expressions)作为操作数，流水线表达式指定了输入文档的转变。表达式是文档结构的，并且可以包含其他的表达式。

​	表达式只可以对流水线中当前的文档进行操作，而不能引用其他文档的数据：表达式操作提供在内存中对文档的转变。

​	通常表达式是无状态的，只有在聚合的过程中才会运算表达式：除了累加表达式。

​	**表达式对象:**

```js
{ <field1>: <expression1>, ... }
```

### Aggregation Pipeline Behaviors 聚合流水线的行为

​	在MongoDB里，如果对一个集合进行聚合命令操作，在逻辑上会将整个集合传进聚合流水线里。所以要尽可能的优化操作，使用以下策略避免扫描集合所有的文档：

**Pipeline operators and indexes 流水线运算符和索引**

[`$match`](https://docs.mongodb.com/manual/reference/operator/aggregation/match/#pipe._S_match)和[`$sort`](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/#pipe._S_sort)管道运算符可以利用索引，当它们出现在流水线的第一个阶段。

管道运算符 [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear) 可以利用地理位置索引。 当使用该运算符时，它必须出现在流水线的第一个阶段。

**Early filtering 提前过滤**

​	如果流水线操作只需要集合里的数据的子集，可以使用 [`$match`](https://docs.mongodb.com/manual/reference/operator/aggregation/match/#pipe._S_match), [`$limit`](https://docs.mongodb.com/manual/reference/operator/aggregation/limit/#pipe._S_limit), 和 [`$skip`](https://docs.mongodb.com/manual/reference/operator/aggregation/skip/#pipe._S_skip) 来限制最初进入流水线的文档。当**$match**操作放在流水线的开头，它就会只扫描集合中匹配的文档。

### Aggregation Pipeline Optimization 聚合流水线优化

聚合流水线操作有优化阶段，该阶段视图重塑管道来提高性能。

#### Projection Optimization投影优化

聚合流水线可以确定是否只需要文档字段的一个子集来计算结果。如果只需要子集，那么流水线会只使用需要的字段，大量减少了通过流水线的数据。

#### Pipeline Sequence Optimization 流水线顺序优化

##### \$project 或者 \$addFields + \$match 顺序优化

对于一个映射阶段([`$project`](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#pipe._S_project)或者[`$addFields`](https://docs.mongodb.com/manual/reference/operator/aggregation/addFields/#pipe._S_addFields))后跟着`$match`阶段的流水线, MongoDB会把`$match`阶段的所有不需要映射阶段的值进行计算的过滤操作放到一个新的`$match`阶段，这个新阶段会在映射阶段之前。

如果流水线中有多个`$project`或`$match`阶段，MongoDB会对每个`$match`阶段进行优化操作。

例如：

```js
{ $addFields: {
    maxTime: { $max: "$times" },
    minTime: { $min: "$times" }
} },
{ $project: {
    _id: 1, name: 1, times: 1, maxTime: 1, minTime: 1,
    avgTime: { $avg: ["$maxTime", "$minTime"] }
} },
{ $match: {
    name: "Joe Schmoe",
    maxTime: { $lt: 20 },
    minTime: { $gt: 5 },
    avgTime: { $gt: 7 }
} }
```

=>

```js
{ $match: { name: "Joe Schmoe" } },
{ $addFields: {
    maxTime: { $max: "$times" },
    minTime: { $min: "$times" }
} },
{ $match: { maxTime: { $lt: 20 }, minTime: { $gt: 5 } } },
{ $project: {
    _id: 1, name: 1, times: 1, maxTime: 1, minTime: 1,
    avgTime: { $avg: ["$maxTime", "$minTime"] }
} },
{ $match: { avgTime: { $gt: 7 } } }
```

##### \$sort + \$match Sequence Optimization

​	如果有这样的顺序，MongoDB会把`$match`放到前面。

例如：

```js
{ $sort: { age : -1 } },
{ $match: { status: 'A' } }
```

=>

```js
{ $match: { status: 'A' } },
{ $sort: { age : -1 } }
```

##### `$redact`+`$match` Sequence Optimization

```js
{ $redact: { $cond: { if: { $eq: [ "$level", 5 ] }, then: "$$PRUNE", else: "$$DESCEND" } } },
{ $match: { year: 2014, category: { $ne: "Z" } } }
```

=>

```js
{ $match: { year: 2014 } },
{ $redact: { $cond: { if: { $eq: [ "$level", 5 ] }, then: "$$PRUNE", else: "$$DESCEND" } } },
{ $match: { year: 2014, category: { $ne: "Z" } } }
```

##### `$project` + `$skip` Sequence Optimization

```js
{ $sort: { age : -1 } },
{ $project: { status: 1, name: 1 } },
{ $skip: 5 }
```

=>

```js
{ $sort: { age : -1 } },
{ $skip: 5 },
{ $project: { status: 1, name: 1 } }
```

#### Pipeline Coalescence Optimization 流水线合并优化

​	优化会尽可能将流水线阶段与前一个阶段合并。通常来讲，合并优化会在重排流水线阶段之后进行。

##### `$sort` + `$limit` 的合并

```js
{ $sort : { age : -1 } },
{ $project : { age : 1, status : 1, name : 1 } },
{ $limit: 5 }
```

=>

```js
{
    "$sort" : {
       "sortKey" : {
          "age" : -1
       },
       "limit" : NumberLong(5)
    }
},
{ "$project" : {
         "age" : 1,
         "status" : 1,
         "name" : 1
  }
}
```

##### `$limit`+`$limit` 的合并

```js
{ $limit: 100 },
{ $limit: 10 }
```

=>

```js
{ $limit: 10 }
```

##### `$skip`+`$skip` 的合并

```js
{ $skip: 5 },
{ $skip: 2 }
```

=>

```js
{ $skip: 7 }
```

##### `$match`+`$match` 的合并

```js
{ $match: { year: 2014 } },
{ $match: { status: "A" } }
```

=>

```js
{ $match: { $and: [ { "year" : 2014 }, { "status" : "A" } ] } }
```

##### `$lookup`+`$unwind` 的合并

```js
{
  $lookup: {
    from: "otherCollection",
    as: "resultingArray",
    localField: "x",
    foreignField: "y"
  }
},
{ $unwind: "$resultingArray"}
```

=>

```js
{
  $lookup: {
    from: "otherCollection",
    as: "resultingArray",
    localField: "x",
    foreignField: "y",
    unwinding: { preserveNullAndEmptyArrays: false }
  }
}
```

### Aggregation Pipeline Limit聚合流水线限制

#### Result Size Restricitons 返回数据大小限制

​	聚合命令可以返回一个游标或者把结果存到一个集合里。无论使用哪种方式，结果集最大为16MB，超过这个大小则报错。这个大小只会限制返回的结果而不会作用在流水线中的某阶段。

#### Memory Restrictions 内存限制

流水线阶段的内存限制为100MB，如果超出则报错。可以通过**allowDiskUse**设置，使聚合流水线可以把数据写到临时文件中，以处理大数据集。

### Aggregation Pipeline and Sharded Collections 聚合流水线和已分片的集合

​	聚合流水线支持在已分片的集合上进行操作。

#### Behavior

​	如果流水线以分片的键`$match`为开始，整个聚合流水线都在该匹配的分片上运行。显然，流水线会被分割，并且会在一个主要的分片上完成组合的工作。

​	对于必须在多个分片上运行的聚合操作，如果操作不需要在数据库的主分片上运行，则这些操作会将结果路由到随机分片以合并结果，以避免重载该数据库的主分片。 [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)阶段和[`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup)阶段要在主分片上运行。

## Map-Reduce

Map-reduce是一种数据处理范例，用于将大量数据压缩为有用的聚合结果。

![image-20190723214451870](http://ww4.sinaimg.cn/large/006tNc79gy1g5a3wgs0hpj31160u0q7i.jpg)

​	在这个Map-Reduce操作中，MongoDB对每个文档都执行map操作(这些文档都符合query的条件)。map函数抛出键值对。对于有多个值的键，MongoDB会使用reduce操作收集和压缩被聚合的数据。然后存到一个集合里。reduce函数的输出还可以输入到finalize函数进一步压缩或处理聚合的返回值。

所有的Map-Reduce函数都是js函数，并且运行在[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)进程上。Map-reduce操作将单个集合的文档作为输入，并且可以在开始映射阶段之前执行任意排序和限制。Map-Reduce可以返回文档或者集合。输入和输出的集合都可能被分片。

### Map-Reduce and Sharded Collections 

#### Sharded Collection as Input

当分片的集合作为Map-Reduce的输入，mongos会自动平行地调度MapReduce操作给每个分片，并且等待所有的分片完成操作。

#### Sharded Collection as Output

如果mapReduce的`out`字段具有分片值，则MongoDB使用`_id`字段作为分片键对输出集合进行分片。

### Map-Reduce Concurrency MapReduce的并发性

Map-Reduce操作由许多任务组成，包括从集合中读取，执行map函数，执行reduce函数，在执行过程中写入临时集合和写入输出集合。

在操作期间，MapReduce会采用下述锁定：

- 读取阶段会采用一个读取锁，每一百个文档锁定一次；
- 插入到临时集合时为一个写操作使用一个写入锁定；
- 如果输出集合不存在，那么输出集合的产出会使用一个写入锁定；
- 如果输出集合存在，那么输出动作也会有写入锁定，这个写入锁定是全局的，并且会阻塞mongod实例的所有操作。

### Perform Incremental Map-Reduce 执行增量的Map-Reduce

​	如果Map-Reduce的数据集持续增长，你也许更想执行增量的Map-Reduce操作而不是每次都对整个数据集执行Map-Reduce操作。

​	要执行增量Map-Reduce：

1. 在当前集合执行Map-Reduce，并输出数据到一个分离的集合；
2. 当你有更多的数据需要处理时，使用以下来运行接下来的Map-Reduce：
   - 指定只匹配新文档的**query**参数作为条件
   - 指定将新结果合并到已存在的输出集合的**reduce**操作

## Aggregation Reference

- [sql-aggregation-comparison](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/)

