Introduce

[TOC]

---

# Introduce

## Document Database 文档型数据库

MongoDB是一个基于文档的通用分布式数据库，专为现代应用程序开发人员和云时代而构建。 没有数据库可以更高效地使用。

MongoDB把每条数据记录存成一个文档，若干文档存放在一起形成一个集合，若干个集合组成文档型数据库。

## Documents 文档

在MongoDB里一条数据记录就是一个BSON文档，文档是由多个键值对组成的数据结构。MongoDB的文档格式上类似于JSON对象。在文档中可以包含其他文档、数组或者文档数组。

![image-20190720170807851](/Users/saltyfish/p/notes/DB/MongoDB/assets/image-20190720170807851.png)

使用文档的优点：

- 文档与很多编程语言的原生数据类型相对应
- 嵌入式的文档和数组减少了很多表连接的代价
- 动态模式支持流畅的多态性

**文档结构**

> 文档由若干个键值对来组成。值可以是BSON支持的任何数据类型，包括其他文档、数组和文档数组。

**字段名称**

> 字段名是一个字符串。有以下限制：
>
> - \_id保留用作主键；\_id的值可以是出了数组之外的任何数据类型。
> - 字段名不能包含 **null** 字符。
> - 高层及的字段名不能用 **$** 开头。

**点表示法**

MongoDB使用点表示法来访问数组的元素和访问值为嵌入文档的字段。

例如：

> 数组：
>
> ```js
> <array>.<index>
> ```
>
> 
>
> 嵌入式文档：
>
> ```js
> <embedded document>.<field>
> ```
>
> 

**文档的限制**

大小限制为16MB; MongoDB会维护插入时字段的顺序，但\_id会一直是第一个字段，并且当有字段名被重命名时，会重新排序。

**\_id字段**

> - 当创建集合时，MongoDB会默认给\_id创建一个无重复的索引；
> - \_id字段默认是第一个字段，如果服务收到一个首字段不是\_id的文档，服务会把它移到第一个；
> - \_id的值可能是除数组之外的任何BSON的数据类型。

### BSON

> BSON是一种二进制序列化格式，在MongoDB中用于存储文档和远程过程调用。

BSON的数据类型有两种识别方式，分别是整数型标识符和字符串标识符。具体可查 [BSON数据类型标识符](https://docs.mongodb.com/manual/reference/bson-types/)

**ObjectId**

ObjectIds体积很小，且近似唯一，可以很快的生成和排序。 ObjectId值由12个字节组成，其中前四个字节是反映ObjectId创建的时间戳。

> 若插入一个没有\_id字段的文档，MongoDB驱动会自动生成一个ObjectId来填充\_id。

### Comparision/Sort Order 比较/排序顺序

当比较不同的BSON数据类型时，MongoDB有一个比较顺序，比如先比较Minkey。[比较顺序](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/)

**数字类型**

MongoDB为了比较，会把某些类型视为等效的类型。例如，数字类型在比较之前转换。

**字符串**

MongoDB在比较字符串时默认会将字符串转变为二进制来比较。

也可以使用 [collation](https://docs.mongodb.com/manual/reference/collation/); 

> **Collation**
>
> Collation特性允许MongoDB的用户根据不同的语言定制排序规则。

**数组**

对于数组，小于比较或升序排序比较数组的最小元素，大于比较或降序排序比较数组的最大元素。

**对象**

MongoDB用一下的顺序来比较对象：

1. 以键值对在对象中出现的顺序来递归地比较；
2. 比较字段名；
3. 如果字段名都相等，则比较字段值；
4. 如果该键值对相等，则比较下一个键值对。

## Collections 集合

MongoDB存储BSON格式的文档到集合里，集合存在数据库里。

![A collection of MongoDB documents.](/Users/saltyfish/p/notes/DB/MongoDB/assets/collection.svg)

### Views 视图

MongoDB可以从已存在集合或者其他视图创建只读的视图。

**视图的行为**

- 只读

- 索引和排序

  > - 视图使用底层集合的索引。
  > - 由于视图使用底层集合的索引，所以不能直接在视图上创建、删除或者重建构建索引，也不能在视图上获取索引列表。
  > - 不能在视图上进行自然($natural)排序

- 不可以改名

### Capped Collections 固定集合

固定集合是一个大小固定的集合，支持基于插入顺序的高吞吐率读写操作。固定集合可以看成一个循环队列。当固定集合已满，当插入新的文档时，会将最早插入的文档覆盖。

* 插入有序

  > 固定集合维护插入的顺序，因此查询有插入顺序的文档时不再需要用索引。没有索引的开销，固定集合可以支持更高的插入吞吐率。

* 自动删除最早插入的文档

* _id 索引

  > 固定集合有\_id字段，同时默认给\_id加索引。

