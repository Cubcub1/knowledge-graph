### 索引的类型
普通索引、唯一索引、主键索引、组合索引、全文索引

#### 创建索引
CREATE [UNIQUE|FULLLTEXT] INDEX index_name ON table_name(column_name(length));
or
ALTER TABLE table_name ADD [UNIQUE|FULLLTEXT] INDEX index_name (column(length));

#### 删除索引
DROP INDEX index_name ON talbe_name;
or
ALTER TABLE table_name DROP INDEX index_name;

#### 索引分析
show index_name fron table_name;

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202210141756206.png)

Non_unique : 非唯一索引，如果是0，代表唯一的，也就是说如果该列索引中不包括重复的值则为0 否则为1
Seq_in_index : 索引中该列的位置，从1开始,如果是组合索引 那么按照字段在建立索引时的顺序排列
Collation : 列是以什么方式存储在索引中的。可以是A或者NULL，B 树索引总是A，排序的
Sub_part : 是否列的部分被索引，如果只是前100行索引，就显示100，如果是整列，就显示NULL
**Cardinality** : 

#### Cardinality 解析
> innodb 优化器是根据这个值来判断是否使用这个索引

该字段不是实时更新的，如果手动更新，使用analyze

##### Innodb存储引擎中Cardinality的策略
-   表中`1/16`的数据发生变化
-   InnoDB存储引擎的计数器`stat_modified_conter`>2000000000

更新逻辑
-   B 树索引中叶子节点数量，记做`A`
-   **随机**取得B 树索引中的`8`个叶子节点。统计每个页不同的记录个数，分别为p1-p8
-   根据采样信息得到Cardinality的预估值：`(p1 p2 p3 ... p8)*A/8`

### 联合索引

####  什么时候需要创建联合索引

- **减少开销** 建一个联合索引(col1,col2,col3)，实际相当于建了(col1),(col1,col2),(col1,col2,col3)三个索引。每多一个索引，都会增加写操作的开销和磁盘空间的开销。对于大量数据的表，使用联合索引会大大的减少开销！
- **覆盖索引** 对联合索引(col1,col2,col3)，如果有如下的sql: select col1,col2,col3 from test where col1=1 and col2=2。那么MySQL可以直接通过遍历索引取得数据，而无需回表，这减少了很多的随机io操作。减少io操作，特别的随机io其实是dba主要的优化策略。所以，在真正的实际应用中，覆盖索引是主要的提升性能的优化手段之一。
- **效率高** 索引列越多，通过索引筛选出的数据越少。有1000W条数据的表，有如下sql:select from table where col1=1 and col2=2 and col3=3,假设假设每个条件可以筛选出10%的数据，如果只有单值索引，那么通过该索引能筛选出1000W10%=100w条数据，然后再回表从100w条数据中找到符合col2=2 and col3= 3的数据，然后再排序，再分页；如果是联合索引，通过索引筛选出1000w10% 10% *10%=1w

#### 为什么不对表中的每一列创建一个索引
- 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加。
- 索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。
- 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。

强制使用某个索引
EXPLAIN SELECT * FROM t_index FORCE INDEX(a) WHERE a = 'a' AND b = 'b' AND c = 'c' ;

#### 索引的适用场景
-   匹配全值

对索引中所有列都指定具体值，即是对索引中的所有列都有等值匹配的条件。

-   匹配值的范围查询

对索引的值能够进行范围查找。

-   匹配最左前缀

仅仅使用索引中的最左边列进行查询，比如在 col1 col2 col3 字段上的联合索引能够被包含 col1、(col1 col2)、（col1 col2 col3）的等值查询利用到，可是不能够被 col2、（col2、col3）的等值查询利用到。 最左匹配原则可以算是 MySQL 中 B-Tree 索引使用的首要原则。

-   仅仅对索引进行查询

当查询的列都在索引的字段中时，查询的效率更高，所以应该尽量避免使用 select *，需要哪些字段，就只查哪些字段。

-   匹配列前缀

仅仅使用索引中的第一列，并且只包含索引第一列的开头一部分进行查找。

-   能够实现索引匹配部分精确而其他部分进行范围匹配
-   如果列名是索引，那么使用 column_name is null 就会使用索引，例如下面的就会使用索引：

```sql
explain select * from t_index where a is null \G
```

-   经常出现在关键字order by、group by、distinct后面的字段
-   在union等集合操作的结果集字段
-   经常用作表连接的字段
-   考虑使用索引覆盖，对数据很少被更新，如果用户经常值查询其中你的几个字段，可以考虑在这几个字段上建立索引，从而将表的扫描变为索引的扫描

#### 索引失效情况

-   以%开头的 like 查询不能利用 B-Tree 索引，执行计划中 key 的值为 null 表示没有使用索引
-   数据类型出现隐式转换的时候也不会使用索引，例如，`where 'age'=30`
	因为age本身是char类型，等式右边是int型，隐式转换等于concat(age)=30。此时等同于对索引列做函数运算
-   对索引列进行函数运算，原因同上
-   正则表达式不会使用索引
-   字符串和数据比较不会使用索引
-   复合索引的情况下，假如查询条件不包含索引列最左边部分，即不满足最左原则 leftmost，是不会使用复合索引的
-   如果 MySQL 估计使用索引比全表扫描更慢，则不使用索引
-   用 or 分割开的条件，如果 or 前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到
-   使用负向查询（not ，not in， not like ，<> ,!= ,!> ,!< ） 不会使用索引
