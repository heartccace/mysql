<u>MySql结构</u>

![](https://github.com/heartccace/mysql/blob/master/images/mysql结构.jpg))

*SQL执行原理*

1. select执行过程

   - 通过连接器与数据库创建连接（查询连接状态 show processlist）
   - 首先查询缓存（缓存以key为sql语句，value为结果集存储数据）命中则返回结果，否则通过分析器（对词法、语法进行分析），完成分析器后再经优化器对语句进行优化，最后通过执行器将结果查询（缓存）并返回。

2.  update执行流程（update T set c=c+1 where ID=2;）

   - 执行器先找引擎取 ID=2 这一行。 ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这
     行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然
     后再返回。
   - 执行器拿到引擎给的行数据，把这个值加上 1 ，比如原来是 N ，现在就是 N+1 ，得到新的一行数据，再调用引擎接口写入这行新数据。
   - 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
   - 执行器生成这个操作的 binlog ，并把 binlog 写入磁盘。
   - 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（ commit ）状态，更新完成。

   

 SQL日志

 	1. redo log（重做日志：InnoDB特有的） WAL(Write Ahead Loging：先将操作记录到日志，等适当时时更新到磁盘)
	2. redo log 是 InnoDB 引擎特有的； binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
	3. redo log 是物理日志，记录的是 “ 在某个数据页上做了什么修改 ” ； binlog 是逻辑日志，记录的
    是这个语句的原始逻辑，比如 “ 给 ID=2 这一行的 c 字段加 1 ” 。
	4. redo log 是循环写的，空间固定会用完； binlog 是可以追加写入的。 “ 追加写 ” 是指 binlog 文件
    写到一定大小后会切换到下一个，并不会覆盖以前的日志









、-

