---
  title: Mongodb遇到的坑
  date: 2014-01-12 12:39:04
  type: "tags"
---

> MongoDB 是目前炙手可热的 NoSQL 文档型数据库，它提供的一些特性很棒：如自动 failover 机制，自动 sharding，无模式 schemaless，大部分情况下性能也很棒。但是薄荷在深入使用 MongoDB 过程中，遇到了不少问题，下面总结几个我们遇到的坑

#### MongoDB Key的插入坑
也就是说在插入字段时属性一定不要带着"a.b.c"

```
  db.users.insert({"a.b.c": 1})
  can't have . in field names [a.b.c] at src/mongo/shell/collection.js:155
```

但是在更新时需要注意

```
  db.users.update({_id: ObjectId("5722ac35e25a087137d138e3")},{$set:{ "a.b.c" : 1 }})
  {
  "_id" : ObjectId("5722ac35e25a087137d138e3"),
  "a_b_c" : 1,
  "a" : 1,
  "x" : {
    "b" : {
      "c" : 1
    }
  }
  }
  都转化成对象了
```

`另外注意一点，当存在属性a再去set a.b.c 是不成立的`

`注意几点： insert添加的属性中包含.是不合法的， update更新的hash里面的key中带有.也是不合法的`

```
db.users.update({_id: ObjectId("5722ac35e25a087137d138e3")},{$set:{ "hash" : {"e.c.v.b": 1} }})
WriteResult({
  "nMatched" : 0,
  "nUpserted" : 0,
  "nModified" : 0,
  "writeError" : {
    "code" : 57,
    "errmsg" : "The dotted field 'e.c.v.b' in 'hash.e.c.v.b' is not valid for storage."
  }
})
```

#### MongoDB的锁的问题
    MongoDB的锁机制和一般关系数据库如 MySQL（InnoDB）, Oracle 有很大的差异，InnoDB 和 Oracle 能提供行级粒度锁，而 MongoDB 只能提供 库级粒度锁，这意味着当 MongoDB 一个写锁处于占用状态时，其它的读写操作都得干等。

    初看起来库级锁在大并发环境下有严重的问题，但是 MongoDB 依然能够保持大并发量和高性能，这是因为 MongoDB 的锁粒度虽然很粗放，但是在锁处理机制和关系数据库锁有很大差异，主要表现在：
    MongoDB 没有完整事务支持，操作原子性只到单个 document 级别，所以通常操作粒度比较小；
    MongoDB 锁实际占用时间是内存数据计算和变更时间，通常很快；
    MongoDB 锁有一种临时放弃机制，当出现需要等待慢速 IO 读写数据时，可以先临时放弃，等 IO 完成之后再重新获取锁。

    通常不出问题不等于没有问题，如果数据操作不当，依然会导致长时间占用写锁，比如下面提到的前台建索引操作，当出现这种情况的时候，整个数据库就处于完全阻塞状态，无法进行任何读写操作，情况十分严重。

    解决问题的方法，尽量避免长时间占用写锁操作，如果有一些集合操作实在难以避免，可以考虑把这个集合放到一个单独的 MongoDB 库里，因为 MongoDB 不同库锁是相互隔离的，分离集合可以避免某一个集合操作引发全局阻塞问题

#### 建索引导致数据库阻塞

    上面提到了 MongoDB 库级锁的问题，建索引就是一个容易引起长时间写锁的问题，MongoDB 在前台建索引时需要占用一个写锁（而且不会临时放弃），如果集合的数据量很大，建索引通常要花比较长时间，特别容易引起问题。
    解决的方法很简单，MongoDB 提供了两种建索引的访问，一种是 background 方式，不需要长时间占用写锁，另一种是非 background 方式，需要长时间占用锁。使用 background 方式就可以解决问题。 例如，为超大表 posts 建立索引， 千万不用使用

```
  db.posts.ensureIndex({user_id: 1})
```
而应该使用

```
  db.posts.ensureIndex({user_id: 1}, {background: 1})
```

#### 不合理使用嵌入 embed document

    embed document 是 MongoDB 相比关系数据库差异明显的一个地方，可以在某一个 document 中嵌入其它子 document，这样可以在父子 document 保持在单一 collection 中，检索修改比较方便。

    比如薄荷的应用情景中有一个 Group document，用户申请加入 Group 建模为 GroupRequest document，我们最初的时候使用 embed 方式把 GroupRequest 放置到 Group 中。 Ruby 代码如下所示（使用了 Mongoid ORM）:

```
class Group
  include Mongoid::Document
  ...
  embeds_many :group_requests
  ...
end

class GroupRequest
  include Mongoid::Document
  ...
  embedded_in :group
  ...
end
```

    这个使用方式让我们掉到坑里了，差点就爬不出来，它导致有接近两周的时间系统问题，高峰时段常有几分钟的系统卡顿，最严重一次甚至引起 MongoDB 宕机。

    仔细分析后，发现某些活跃的 Group 的 group_requests 增加（当有新申请时）和更改（当通过或拒绝用户申请时）异常频繁，而这些操作经常长时间占用写锁，导致整个数据库阻塞。原因是当有增加 group_request 操作时，Group 预分配的空间不够，需要重新分配空间（内存和硬盘都需要），耗时较长，另外 Group 上建的索引很多，移动 Group 位置导致大量索引更新操作也很耗时，综合起来引起了长时间占用锁问题。

    解决问题的方法，说起来也简单，就是把 embed 关联更改成的普通外键关联，就是类似关系数据库的做法，这样 group_request 增加或修改都只发生在 GroupRequest 上，简单快速，避免长时间占用写锁问题。当关联对象的数据不固定或者经常发生变化时，一定要避免使用 embed 关联，不然会死的很惨。

#### 不合理使用 Array 字段

    MongoDB 的 Array 字段是比较独特的一个特性，它可以在单个 document 里存储一些简单的一对多关系。

    薄荷有一个应用情景使用遇到严重的性能问题，直接上代码如下所示：

```
class User
  include Mongoid::Document
  ...
  field :follower_user_ids, type: Array, default: []
  ...
end
```

###### 参考资料:

- [MongoDB 锁机制详解](https://docs.mongodb.org/manual/faq/concurrency/)
- [MongoDB 建立索引操作文档](https://docs.mongodb.org/manual/core/index-creation/)
