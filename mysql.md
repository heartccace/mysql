

##### show status

返回一些计数器，既有服务器级别的全局计数器，也有基于某个连接的会话级别的计数器。

##### show global status

可以查看服务器级别的从服务器启动是开始计算的查询次数

#####  show processlist  \g

查看线程状态

##### long_query_time 

慢查询，默认为0，根据实际业务需要修改值

##### slow_query_log 

慢查询开启状态

##### slow_query_log_file

慢查询日志存放位置

#### 索引

> CREATE TABLE People (
> 	last_name varchar(50) not null,
>
> ​	first_name varchar(50) not null,
>
> ​	dob date not null,
>
> ​	gender enum('m','f') not null,
>
> ​	key(last_name,first_name,dob)	
> )



##### 全值匹配

全值匹配指的是和索引中的所有列进行匹配。使用所有索引创建的值

##### 匹配最左前缀

前面索引查找所有姓为Allen的人，只使用索引的第一列

##### 匹配列前缀

也可以匹配某一列的值的开头部分。例如前面提到的索引可用于查找所有以J开头的姓的人。这里也只使用索引的第一列

##### 匹配范围值

##### 精确匹配某一列并范围匹配到另一列

第一列全匹配，第二列范围匹配



##### 关于B-Tree的一些限制

- 如果不是按照索引的最左列开始，则无法使用索引。
- 不能跳过索引中的列
- 如果查询中有某个列的范围查询，则最右边所有列都无法使用索引优化查询

##### 哈希索引

哈希索引基于哈希表实现，只有精确匹配到索引所有列的查询才有效。对于每一行数据，存储引擎都会对所有的索引列计算出一个hash码。hash码是一个较小值，并且不同键值的行计算出来的哈希码也不一样。哈希索引将所有的哈希码存储在索引中，同时在哈希表中保存指向每个数据行的指针。

#### 高性能索引策略

- 独立的列
- 前缀索引和索引选择性
- 多列索引
- 选择合适的列作为索引   