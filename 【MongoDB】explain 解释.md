https://juejin.cn/post/6918927746529918984

### explain 命令

〉db.collection.find().explain("executionStats")

#### explain 的三种模式
- queryPlanner： 不会真正的执行查询，只是分析查询，选出winning plan
- executionStats： 返回winning plan 的关键数据
- allPlansExecution：执行所有的plans

### explain 分析
####  executionStats 字段
executionTimeMillis ：查询语句总耗时。上述查询语句 5584702 条一共耗时 14920 ms
totalKeysExamined：索引扫描条目。 上述结果为 0，说明本次查询可能没有用到索引
totalDocsExamined：文档扫描条数。 上述结果为 5584702，说明本次查询把文档全部扫描了一遍
nReturned：返回结果条数。 上述结果返回10条
#### executionStages
- **nReturned**：**查询返回的条目**
- **totalKeysExamined**：**索引扫描条目**
- **totalDocsExamined**：**文档扫描条目**
#### stage状态
- stage：查询计划类型
- inputStage：用来描述子stage，并且为其父stage提供文档和索引关键字

> COLLSCAN：全表扫描  
> IXSCAN：索引扫描  
> FETCH：根据索引去检索指定document  
> SHARD_MERGE：将各个分片返回数据进行merge  
> SORT：表明在内存中进行了排序  
> LIMIT：使用limit限制返回数  
> SKIP：使用skip进行跳过  
> IDHACK：针对_id进行查询  
> SHARDING_FILTER：通过mongos对分片数据进行查询  
> COUNT：利用db.coll.explain().count()之类进行count运算  
> COUNTSCAN： count不使用Index进行count时的stage返回  
> COUNT_SCAN： count使用了Index进行count时的stage返回  
> SUBPLA：未使用到索引的$or查询的stage返回  
> TEXT：使用全文索引进行查询时候的stage返回  
> PROJECTION：限定返回字段时候stage的返回


### 良好的查询

**对于一般查询，建议stage组合(**查询的时候尽可能用上索引**)：
Fetch+IDHACK  
Fetch+ixscan  
Limit+（Fetch+ixscan）  
PROJECTION+ixscan  
SHARDING_FITER+ixscan  
COUNT_SCAN

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202211191126629.png)
