---
layout: post
title: MongoDB Capped Collection 使用
date: 2015-05-5 00:00
tags: [mongodb]
---

## MongoDB Capped Collection 使用

### Capped Collection 简介

- > Capped Collection 是一种特殊的集合，它大小固定，当集合的大小达到指定大小时，新数据覆盖老数据。Capped collections可以按照文档的插入顺序保存到集合中，而且这些文档在磁盘上存放位置也是按照插入顺序来保存的，所以当我们更新Capped collections中文档的时候，更新后的文档不可以超过之前文档的大小，这样话就可以确保所有文档在磁盘上的位置一直保持不变。
- > 由于Capped collection是按照文档的插入顺序而不是使用索引确定插入位置，这样的话可以提高增添数据的效率。MongoDB的操作日志文件oplog.rs就是利用Capped Collection来实现的。
- > 除此之外，Capped Collection还有以下的一些特点，首先是不允许删除，但是可以调用drop（）删除集合中的所有行，不许删除的原因也是为了保持每个文档在磁盘上的位置不变。在32位机器上一个capped collection的最大值约482.5M，64位上没有限制系统文件大小限制。不可以对 Capped Collection 进行分片，在 2.2 版本以后，创建的Capped Collection 默认在  _id 字段上创建索引，而在 2.2 版本或以前没有。
- > Capped Collection主要用于存储日志信息和缓存一些少用的文档。

### Capped Collection 使用

#### 创建Capped Collection

```
> db.createCollection("mycoll",{capped:true,size:1024,max:10})
{ "ok" : 1 }
> for(var i=1;i<1000;i++) db.mycoll.save({id:i,name:"Hello"})
> db.mycoll.count()
> 10
> db.mycoll.find()
{ "_id" : ObjectId("5274b854f007ff8bb84f6347"), "id" : 990, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f6348"), "id" : 991, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f6349"), "id" : 992, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f634a"), "id" : 993, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f634b"), "id" : 994, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f634c"), "id" : 995, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f634d"), "id" : 996, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f634e"), "id" : 997, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f634f"), "id" : 998, "name" : "Hello" } { "_id" : ObjectId("5274b854f007ff8bb84f6350"), "id" : 999, "name" : "Hello" }
> db.mycoll.isCapped()
> true
> db.mycoll.stats()
{
"ns" : "test.mycoll",
"count" : 0,
"size" : 0,
"storageSize" : 4096,
"numExtents" : 1,
"nindexes" : 1,
"lastExtentSize" : 4096,
"paddingFactor" : 1,
"systemFlags" : 1,
"userFlags" : 0,
"totalIndexSize" : 8176,
"indexSizes" : {
"_id_" : 8176
},
"capped" : true,
"max" : NumberLong("9223372036854775807"),
"ok" : 1
}
```
- > 上面个创建了一个Capped Collection，占用1024个字节，最多10个文档，然后向其中插入了1000条数据通过db.collection.isCapped() 和 db.mycoll.stats()命令可以查看一个集合是否是 Capped Collection 。

`注意： `

- a.创建固定集合不像普通集合，固定集合需要显示的创建使用
- b.创建固定集合必须指定其大小否则会报错：
- c.创建集合的时候Max属性是可选的，文档数量限制是在容量没满的时候进行淘汰，要是满了，就根据容量限制来替换数据。例如上面的例子我们设置文档数量限制是10这样的话在没有达到容量上限的时候，最多只能存储10个文档。

#### 查看索引
  在2.2版本以后，创建的Capped Collection 默认在  _id 字段上创建索引。

#### 尝试去更新一个文档
  更新后的文档大小值不能超过原有空间，否则更新失败。

#### 尝试去删除一个文档
  使用Capped Collection 不能删除一个文档。
  可以删除集合

#### 普通集合转成固定集合
  首先我们创建了一个普通集合，然后使用命令 db.runCommand({convertToCapped:'mycoll',size:10000,max:3})将其转化为一个Capped Collection

#### 试用领域
  高写入数据不更新数据，只需要保存最近的数据
  日志领域

#### 替代方案
  可以使用mongDB的TTL方案