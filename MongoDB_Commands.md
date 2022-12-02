# MongoDB 基本指令操作
###### tags: `DB` `NoSQL`
### MongoDB Docker基本安裝
下載鏡像
```
docker pull mongo
```

在本地端建立一個可以儲存DB資料的資料夾 mongodb_data
```
mkdir mongodb_data
```
啟動並轉port
```
docker run --name test_mongo -v $(pwd)/mongodb_data:/data/db -d -p 27017:27017 --rm mongo
```
進入docker
```
docker -it test_mongo bash
```

### 進入DB命令列(Shell)
```
mongosh
```
### 基本指令操作 CRUD

#### 顯示目前操作DB
```
testdb> db
testdb
```
#### 檢視DB
```
testdb> show dbs
admin    40.00 KiB
config  108.00 KiB
local    40.00 KiB
testdb   72.00 KiB
```
#### 切換DB(當沒有該DB時自動創建)
```
admin> use testdb
switched to db testdb
```
#### 刪除DB
```
testdb> use testdb2
switched to db testdb2
testdb2> db.dropDatabase()
{ ok: 1, dropped: 'testdb2' }
```
#### 創建集合collections
```
testdb> db.createCollection('testCollect2')
{ ok: 1 }
```
#### 查看已有集合
可使用 show collections 或 show tables
```
testdb> show collections
testCollect
testCollect2
testdb> show tables
testCollect
testCollect2
```
在 MongoDB 中，你不需要創建集合。當你插入一些資料时，MongoDB會自動創建集合。
```
testdb> db.testCollect3.insertOne({'name':'kelly'})
{
  acknowledged: true,
  insertedId: ObjectId("6388575272d61551eaf91e78")
}
testdb> show tables
testCollect
testCollect2
testCollect3
```
#### 刪除集合
> drop()
```
testdb> show tables
testCollect
testCollect2
testCollect3
testdb> db.testCollect3.drop()
true
testdb> show tables
testCollect
testCollect2
```
#### 新增資料
新增一筆資料 
> insertOne()
```
testdb> db.rooms.insertOne(
 	{name:"單人房",price:1000,rating:4.5}
)
{
  acknowledged: true,
  insertedId: ObjectId("63885aec72d61551eaf91e79")
}
```
新增多筆資料
> insertMany()
```
testdb> db.rooms.insertMany(
     [
         {
             "rating": 4.3,
             "price": 1500,
             "name": "豪華單人房"
         },
         {
             "rating": 4.8,
             "price": 2000,
             "name": "雙人房"
         }
     ]
)
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("63885b6d72d61551eaf91e7a"),
    '1': ObjectId("63885b6d72d61551eaf91e7b")
  }
}
```
也可以把資料存入到一個變數中
```
testdb> document=({'name':'test nosql'})
{ name: 'test nosql' }

testdb> db.rooms.insertOne(document)
{
  acknowledged: true,
  insertedId: ObjectId("63885e4f2facbdf49a82817d")
}
```
#### 更新資料
用于更新已存在的資料。語法格式如下：
> update()
* query : update的查詢條件，類似sql update查詢内where後面的。
* update : update的對象和一些更新的操作符（如$,$inc...）等，也可以理解為sql update查詢内set後面的
* upsert : 可選，这個参数的意思是，如果不存在update的紀錄，是否插入objNew,true為插入，默認是false，不插入。
* multi : 可選，mongodb 默認是false,只更新找到的第一筆紀錄，如果这這個参数為true,就把按條件查出来多條紀錄全部更新。
* writeConcern :可選，抛出異常的级别。
```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```
更新剛剛新增的 {name:'test nosql'}
```
testdb> db.rooms.update({'name':'test nosql'},{$set:{'name':'test nosql update'}})
DeprecationWarning: Collection.update() is deprecated. Use updateOne, updateMany, or bulkWrite.
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
testdb> db.rooms.find().pretty()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1000,
    rating: 4.5
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7b"),
    rating: 4.8,
    price: 2000,
    name: '雙人房'
  },
  {
    _id: ObjectId("63885e4f2facbdf49a82817d"),
    name: 'test nosql update'
  }
]
```
可以看到名稱(name)由原来的 "test nosql" 更新為了 "test nosql update"。

以上語句只会修改第一條發現的資料，如果你要修改多條相同的資料，則需要設置 multi 参数為 true。
```
testdb> db.rooms.update({'name':'test nosql'},{$set:{'name':'test nosql update'}},{multi:true})
```
更新一筆資料欄位
> updateOne()
```
testdb> db.rooms.updateOne({"_id":ObjectId("63885e4f2facbdf49a82817d")},{$set:{"name":"test nosql updateOne"}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
testdb> db.rooms.find().pretty()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1000,
    rating: 4.5
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7b"),
    rating: 4.8,
    price: 2000,
    name: '雙人房'
  },
  {
    _id: ObjectId("63885e4f2facbdf49a82817d"),
    name: 'test nosql updateOne'
  }
]
```
更新多個值(filter 篩選 >$set 更換設定)
> updateMany() 
```
testdb> db.rooms.updateMany(
  {rating:{$gt:4.5}},
  {$set:{price:300}}
)
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
testdb> db.rooms.find().pretty()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1000,
    rating: 4.5
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7b"),
    rating: 4.8,
    price: 300,
    name: '雙人房'
  },
  {
    _id: ObjectId("63885e4f2facbdf49a82817d"),
    name: 'test nosql updateOne'
  }
]
```
根據過濾器替換集合中的單個資料
> replaceOne()
* filter: 指定一個空資料以替換集合中返回的第一筆資料
* replacement: 替換資料
* upsert: 可選，这個参数的意思是，如果不存在update的紀錄，是否插入objNew,true為插入，默認是false，不插入。
* writeConcern: 可選的。表達書面關切的文件。省略使用默認的寫關注。
* collation: 可選的。指定用於操作的排序規則。
* hint: 可選的。指定用於支持過濾器的索引的文檔或字符串。
```
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     hint: <document|string>                   // Available starting in 4.2.1
   }
)
```
更新單人房價格與評分
```
testdb> db.rooms.replaceOne({'name':'單人房'},{'name':'單人房','price':1100, 'rating':4.4})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
testdb> db.rooms.find().pretty()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1100,
    rating: 4.4
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  }
]
```


#### 刪除資料
刪除一筆資料
> deleteOne()
```
testdb> db.rooms.deleteOne({"name":"test nosql updateOne"})
{ acknowledged: true, deletedCount: 1 }
testdb> db.rooms.find().pretty()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1000,
    rating: 4.5
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7b"),
    rating: 4.8,
    price: 300,
    name: '雙人房'
  }
]
```
刪除多筆資料
> deleteMany()
```
testdb> db.rooms.find()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1000,
    rating: 4.5
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7b"),
    rating: 4.8,
    price: 300,
    name: '雙人房'
  },
  { _id: ObjectId("63886f092facbdf49a82817e"), name: '雙人房' }
]
testdb> db.rooms.deleteMany({'name':'雙人房'})
{ acknowledged: true, deletedCount: 2 }
testdb> db.rooms.find()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1000,
    rating: 4.5
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  }
]
```
#### 查詢資料
MongoDB 查詢資料的語法格式如下：
* query：可選，使用查詢操作符指定查詢條件
* projection：可選，使用投影操作符指定返回的鍵。查詢時返回資料中所有鍵值， 只需省略該參數即可（默認省略）。
```
db.collection.find(query, projection)
```
如果你需要以易讀的方式來讀取數據，可以使用pretty() 方法，語法格式如下：
```
db.col.find().pretty()
```
查詢資料
> find()
```
testdb> db.rooms.find().pretty()
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1100,
    rating: 4.4
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  }
]
```
查詢一筆資料
> findOne()
```
testdb> db.rooms.findOne()
{
  _id: ObjectId("63885aec72d61551eaf91e79"),
  name: '單人房',
  price: 1100,
  rating: 4.4
}
```
**MongoDB 與RDBMS Where 語句比較**
MongoDB中條件操作符有：
* (>) 大於- $gt
* (<) 小於- $lt
* (>=) 大於等於- $gte
* (<= ) 小於等於- $lte

如果你熟悉常規的SQL 數據，通過下表可以更好的理解MongoDB 的條件語句查詢：

| 操作 | 格式 | 範例 | RDBMS中的類似語句 |
| -------- | -------- | -------- | -------- |
| 等於     | {<key>:<value>}| db.col.find({"by":"菜鳥教程"}).pretty()     | where by = '菜鳥教程'|
|小於	|{<key>:{$lt:<value>}}|	db.col.find({"likes":{$lt:50}}).pretty()	|where likes < 50 |
|小於或等於	|{<key>:{$lte:<value>}}|	db.col.find({"likes":{$lte:50}}).pretty()|	where likes <= 50 |
|大於	|{<key>:{$gt:<value>}}	|db.col.find({"likes":{$gt:50}}).pretty()|	where likes > 50 |
|大於或等於	|{<key>:{$gte:<value>}}	|db.col.find({"likes":{$gte:50}}).pretty()|	where likes >= 50 |
|不等於	|{<key>:{$ne:<value>}}|	db.col.find({"likes":{$ne:50}}).pretty()|	where likes != 50 |
    
**MongoDB AND 條件**
MongoDB 的find() 方法可以傳入多個鍵(key)，每個鍵(key)以逗號隔開，即常規SQL 的AND 條件。
語法格式如下：
```
db.col.find({key1:value1, key2:value2}).pretty()
```
實例
```
testdb> db.rooms.find({'name':'單人房','rating':4.4})
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1100,
    rating: 4.4
  }
]
```
**MongoDB OR 條件**
MongoDB OR 條件語句使用了關鍵字$or ,語法格式如下：
```
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```
實例
```
testdb> db.rooms.find({$or:[{'name':'單人房'},{'rating':4.3}]})
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1100,
    rating: 4.4
  },
  {
    _id: ObjectId("63885b6d72d61551eaf91e7a"),
    rating: 4.3,
    price: 1500,
    name: '豪華單人房'
  }
]
```
**AND 和OR 聯合使用**
以下實例演示了AND 和OR 聯合使用，類似常規SQL 語句為： 'where rating>4.3 AND (name = '單人房' OR price = 1100)'
```
testdb> db.rooms.find({'rating':{$gt:4.3},$or:[{'name':'單人房'},{'price':1100}]})
[
  {
    _id: ObjectId("63885aec72d61551eaf91e79"),
    name: '單人房',
    price: 1100,
    rating: 4.4
  }
]
```