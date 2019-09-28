<u>MySql结构</u>

![](https://github.com/heartccace/mysql/blob/master/images/mysql结构.jpg))

*SQL执行原理*

1. select执行过程

   	- 通过连接器与数据库创建连接（查询连接状态 show processlist）
   	- 首先查询缓存（缓存以key为sql语句，value为结果集存储数据）命中则返回结果，否则通过分析器（对词法、语法进行分析），完成分析器后再经优化器对语句进行优化，最后通过执行器将结果查询（缓存）并返回。

   

 SQL日志

 1. redo log（重做日志：InnoDB特有的） WAL(Write Ahead Loging：先将操作记录到日志，等适当时时更新到磁盘)

 2. redo log 是 InnoDB 引擎特有的； binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

3. redo log 是物理日志，记录的是 “ 在某个数据页上做了什么修改 ” ； binlog 是逻辑日志，记录的
  是这个语句的原始逻辑，比如 “ 给 ID=2 这一行的 c 字段加 1 ” 。

4. redo log 是循环写的，空间固定会用完； binlog 是可以追加写入的。 “ 追加写 ” 是指 binlog 文件
  写到一定大小后会切换到下一个，并不会覆盖以前的日志

  









、-

