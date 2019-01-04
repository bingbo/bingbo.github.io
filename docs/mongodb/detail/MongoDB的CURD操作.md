这里简单列出主要的读写操作接口。

其他详情完整API可参考官网：https://docs.mongodb.org/manual/reference

## 读操作

### Query

> db.collection.findOne()

返回匹配的文档记录的一条

> db.collection.find()

查看所有匹配的文档记录，可以通过`limits`、`skips`和`sort orders`来影响查询结果。

```node
db.users.find( { age: { $gt: 18 } }, { name: 1, address: 1 } ).limit(5)
```

### Cursors

主要的读方法db.collection.find()其实返回的就是一个游标

> cursor.noCursorTimeout()

可以禁止游标默认在不活跃的情况下10分钟后自动关闭，但这样的话必须手动关闭通过`cursor.close()`。

```node
var myCursor = db.inventory.find().noCursorTimeout();
```

> cursor.next()

获取下一条记录文档

> cursor.objsLeftInBatch()

还有多少文档记录在batch里，即还有多少条记录没有读。

```node
var myCursor = db.inventory.find();
var myFirstDocument = myCursor.hasNext() ? myCursor.next() : null;
myCursor.objsLeftInBatch();
```

> db.serverStatus()

返回数据库的配置状态信息

> db.serverStatus().metrics.cursor

是其中的游标配置信息，结果如下所示：

```node
{
   "timedOut" : <number>
   "open" : {
      "noTimeout" : <number>,
      "pinned" : <number>,
      "total" : <number>
   }
}
```

### Query Optimization

> db.collection.createIndex()

创建索引，包括单独索引和聚合索引等。其中参数keys为1为正排索引，-1为倒排索引。

```node
db.collection.createIndex( { orderDate: 1, zipcode: -1 } )
```

> db.collection.explain()

返回一些查询操作的性能信息，以便优化查询

```node
db.products.explain().remove( { category: "apparel" }, { justOne: true } )
```

## 写操作

### Insert

> db.collection.insertOne()

插入一条单独的文档记录

```javascript
db.users.insertOne(
   {
      name: "sue",
      age: 26,
      status: "pending"
   }
)
```

> db.collection.insertMany()

一次插入多条文档记录

```javascript
db.users.insertMany(
   [
      { name: "sue", age: 26, status: "pending" },
      { name: "bob", age: 25, status: "enrolled" },
      { name: "ann", age: 28, status: "enrolled" }
   ]
)
```

> db.collection.insert()

> 插入新的文档记录到表集合中，可以插入一个文档记录也可以插入多个文档记录。

```javascript
db.users.insert(
   {
      name: "sue",
      age: 26,
      status: "A"
   }
)
```

### Update

> db.collection.updateOne()

更新一个文档记录

```javascript
db.users.updateOne(
   { age: { $lt: 18 } },
   { $set: { status: "reject" } }
)
```

> db.collection.updateMany()

一次更新多个文档记录

```javascript
db.users.updateMany(
   { age: { $lt: 18 } },
   { $set: { status: "reject" } }
)
```

> db.collection.replaceOne()

替换现有的一个文档记录，不能使用更新操作符，且替换匹配的第一条文档记录

```javascript
db.users.replaceOne(
   { name: "sue" },
   { name: "amy", age : 25, score: "enrolled" }
)
```

> update

该方法修改已存在的文档记录，可以指定`multi:true`参数更新多个匹配的文档记录

```node
db.users.update(
   { age: { $gt: 18 } },
   { $set: { status: "A" } },
   { multi: true }
)
```

### Delete

> db.collection.deleteOne()

删除匹配的第一个文档记录

```node
db.users.deleteOne(
   { status: "reject" }
)
```

> db.collection.deleteMany()

删除多条匹配的文档记录

```node
db.users.deleteMany(
   { status: "reject" }
)
```

> db.collection.remove()

删除所有匹配的文档记录，指定`justOne:true`参数可以只删除一条文档记录

```node
db.users.remove(
   { status: "D" }
)
```

### Others

> db.collection.save()

可以更新已存在的文档记录或是插入一个新的记录如果不存在的话。

### Bulk Write

> db.collection.bulkWrite()

可以一次执行多种写操作，每种操作在一个文档记录上是原子操作

```node
db.collection.bulkWrite(
   [
      { insertOne : { "document" : { name : "sue", age : 26 } } },
      { insertOne : { "document" : { name : "joe", age : 24 } } },
      { insertOne : { "document" : { name : "ann", age : 25 } } },
      { insertOne : { "document" : { name : "bob", age : 27 } } },
      { updateMany: {
         "filter" : { age : { $gt : 25} },
         "update" : { $set : { "status" : "enrolled" } }
         }
      },
      { deleteMany : { "filter" : { "status" : { $exists : true } } } }
   ]
)
```
