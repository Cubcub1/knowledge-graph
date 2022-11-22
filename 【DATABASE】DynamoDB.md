
### 什么是DynamoDB

- NoSQL
- 高可用和持久性

### DynamoDB 关键字
- 表 ： 类似mysql中的表
- 项目：表中的一条记录
- 属性：项目拥有的字段 常见是数字和字符串，json
> 一张表的不同项目可以拥有不同属性

### 主键
- 分区键(hash key)：在只有分区键的表中，不允许重复。dynamodb用于分区存储的hash输入
- 分区键(hash key)和排序键(range key)：类比为mysql 的联合唯一索引。分区键用于分区存储的hash输入，排序键用于将同一个分区键的项目排序存储

> 分区键仅支持判等操作。
> 主键必须唯一；数据类型只支持字符串、数字和二进制


### 二级索引
-   全局二级索引 – 分区键和排序键可与基表中的这些键不同的索引。
-   本地二级索引 – 分区键与基表相同但排序键不同的索引。

#### DDL和DML
[dynamodb API](https://docs.aws.amazon.com/zh_cn/amazondynamodb/latest/developerguide/HowItWorks.API.html)



