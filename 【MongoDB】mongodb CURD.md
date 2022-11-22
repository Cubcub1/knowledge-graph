## [mongodb文档](https://www.mongodb.com/docs/manual/tutorial/query-arrays/) 

## Insert

`db.collection.insertOne()`
`db.collection.insertMany()`

`db.collection.bulkWrite()`

## Query

`{ <field> : <value> }`或`{ <field> : { <operator> : <value> } }`

```
**eq**

{<field>:<value>} //{ status: "A" }

**and**
{<f1>:<value>,<f2>:<value>} //{ status: "A", qty: { $lt: 30 } }

**or**
{$or:[{<f1>:<value>},{<f2>:<value>}]} // { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] }

/*----------------------*/

// nested query
{<field.attr>:<value>} // { "size.uom": "in" }

/*----------------------*/

// array query

/*
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
   { item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
   { item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
   { item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
   { item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }
]);
*/

// tags的值完全匹配 ["red", "blank"]
db.inventory.find( { tags: ["red", "blank"] } )

// tags的值包含["red", "blank"]
db.inventory.find( { tags: { $all: ["red", "blank"] } } )

// tags的值包含 "red"
db.inventory.find( { tags: "red" } )

// dim_cm的值至少有一个满足
db.inventory.find( { dim_cm: { $gt: 25 } } )

// dim_cm的值有两个分别满足 或 至少有一个都满足
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )

// dim_cm的值 至少有一个都满足
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )

// dim_cm按照下标取值 满足 > 25
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )

// tags 的长度满足条件
db.inventory.find( { "tags": { $size: 3 } } )

/*----------------------*/

// item 为null，或者不包含item的记录
db.inventory.find( { item: null } )

// item 为null的记录
db.inventory.find( { item : { $type: 10 } } )

// 不包含item的记录
db.inventory.find( { item : { $exists: false } } )

```

## update

`db.collection.updateOne(<filter>, [ { f1:v,f2:v } ] | [ {operator: { f1:v1,f2:v2 }} ])`
`db.collection.updateMany()`

更新语句中 的数组，按照流水线形式顺序执行 
-   [`$addFields`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addFields/#mongodb-pipeline-pipe.-addFields)
-   [`$set`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/set/#mongodb-pipeline-pipe.-set)
-   [`$project`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/project/#mongodb-pipeline-pipe.-project)
-   [`$unset`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/unset/#mongodb-pipeline-pipe.-unset)： 可以用来删除document的字段
-   [`$replaceRoot`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/replaceRoot/#mongodb-pipeline-pipe.-replaceRoot)
-   [`$replaceWith`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/replaceWith/#mongodb-pipeline-pipe.-replaceWith)

