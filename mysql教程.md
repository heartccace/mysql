# 1 mysql教程

## 1.1 MySQL运行原理

访问mysql数据库，需要通过mysql提供的驱动来跟数据库建立连接。建立连接之后我们的代码才能基于这个连接进行增删改查。

数据库连接池：

为了提高应用访问mysql的效率，应用访问数据库时会从连接池里面获取连接，或者新建连接。

![客户端连接池](../mysqldoc/images/客户端连接池.PNG)

同时mysql端也会维护一个连接池,用于管理各种系统跟这台数据库服务器建立的连接

![客户端和mysql连接池](../mysqldoc/images/客户端和mysql连接池.PNG)

## 1.2 mysql执行过程

网络连接必须让线程处理，mysql从网络连接中读取出来一个sql之后，会交给内部的工作线程去处理。mysql提供了一个组件，就是SQL接口（SQL Interface），它是一套执行sql语句的接口，专门用于执行我们发送给mysql的那些增删改查的sql语句。

因此mysql的工作线程接收到sql语句之后，就会转交给SQL接口去执行。

![工作线程](../mysqldoc/images/工作线程.PNG)

SQL接口不能直接处理SQL语句，此时需要查询解析器，查询解析器就是负责对SQL语句进行解析，SQL解析器会对按照SQL语法规则编写的SQL语句进行解析，然后理解这个SQL语句要干什么事情。

![sql解析器](../mysqldoc/images/sql解析器.PNG)

解析出来的SQL会交给查询优化器，查询优化器会根据解析出来的SQL选择最佳的方案。

![优化器](../mysqldoc/images/优化器.PNG)

优化器找到最佳优化方案之后调用存储引擎，真正执行SQL。



![存储引擎](../mysqldoc/images/存储引擎.PNG)

存储引擎出则执行SQL，它会按照一定的不走去查询内存缓存数据，更新磁盘数据，查询磁盘数据，等等。

存储引擎是通过执行器进行调用的

![执行器](../mysqldoc/images/执行器.PNG)

## 1..3 InnoDB的设计

InnoDB中有一个非常重要的放在内存的组件，就是缓冲池（Buffer Pool），这里面会缓存很多的数据，以便以后查询的时候，万一缓存池里面有数据，就可以不用去查磁盘了。

![bufferedPool](../mysqldoc/images/bufferedPool.PNG)

引擎要执行更新语句的时候，比如对“id= 10”这行数据。它首先会看看这一行数据是否在缓冲池，如果不在，就会从磁盘加载到缓冲池，而且接着还会对这行记录加独占锁

### 1.3.1 undo日志文件

接着下一步，假设“id=10”这行数据的name原来是”lishi“，现在要更新为“zhangsan”，那么此时我们先要把更新的原来的值“id=10”和“lishi”写入到undo日志文件中。如果要执行一个更新语句，要是它在一个事务中，那么事务提交前都可以对数据进行回滚，也就是把“lishi”回滚到“zhangsan”，所以为了考虑到未来可能要回滚数据，这里会把更新前的值写入到undo日志文件中。

![undolog](../mysqldoc/images/undolog.PNG)

当记录从磁盘加载到缓冲池，对它加锁之后，把旧值写入到undo日志文件中。我们就可以正式更新这行记录了。更新的时候，显示会更新缓冲池中的记录，此时这个数据就是脏数据了（此时数据还没写回到磁盘，内存数据和缓存数据不一致）。

![脏数据](../mysqldoc/images/脏数据.PNG)

### 1.3.2 Redo Log Buffer: 宕机，避免数据丢失

接上图，如果内存的数据进行修改了，但是磁盘上的数据还没修改，那么此时mysql宕机了，必然会导致内存数据丢失，如何处理呢？

这个时候就必须把堆内存所做的修改写入到一个Redo Log Buffer里去，这也是内存里面的一个缓冲区，就是用来存放redo日志的。

所谓的redo日志，就是记录下你对数据做了什么修改。

![redoLog](../mysqldoc/images/redoLog.PNG)

redo日志就是用来在MYSQL突然宕机的时候，恢复更新过的数据。

#### 1.3.2.1 如果事务还没提交，MySQL宕机了怎么办？

执行一个事务，只有当事务提交之后SQL语句才算真正执行完成。

所以目前为止我们还没提交事务，此时mysql崩溃，必然会导致内存数据丢失，同时写入Redo Log Buffer中的redo日志也会丢失。

![redolog丢失](../mysqldoc/images/redolog丢失.PNG)

此时数据丢失重要？

其实是不重要的，因为一条更新语句没提交事务就是没执行成功，此时宕机，导致内存数据丢失，磁盘上的数据依然还提留在原样子。此时你会收到一个数据库异常，当数据库重启后，你会发现数据没有任何变化。

#### 1.3.2.2 提交事务的时候将redo日志写入的磁盘

接着我们想要提交一个事务，此时会根据一定的策略把redo日志从redo log buffer刷入到磁盘上去。

这个策略可以通过innodb_flush_log_at_trx_commit来配置，他又几个取值：

- 当取值为0时：那么提交事务的时候，不会把redo log buffer中的数据刷到磁盘上，此时mysql宕机，内存数据依然会丢失。相当于你事务提交成功了，mysql宕机，导致内存数据和redo日志丢失，如图：

  ![redo策略0](../mysqldoc/images/redo策略0.PNG)

- 当取值为1时：提交事务的时候就必须把redo log从内存刷到磁盘上，只要事务提交成功了，那么redo log就必须在磁盘里了，如图：

  ![redo策略1](../mysqldoc/images/redo策略1.PNG)

​		如果buffer pool中的数据还没刷到磁盘上，内存数据发生修改，可能处于如下状态：

![redolog策略1-1](../mysqldoc/images/redolog策略1-1.PNG)

此时mysql宕机，数据会丢失？

肯定不会，因为redo日志已经记录了更新，所以mysql重启之后，它可以根据redo日志去恢复之前做过的操作。

![redolog策略1-2](../mysqldoc/images/redolog策略1-2.PNG)

- 当取值为2时：提交事务的时候将redo日志写到磁盘文件对应的os cache缓存里面去，而不时直接进入磁盘文件，可能1s后才会把os cache里面的数据写入到磁盘。这种模式下，事务提交之后，redo log可能仅仅提留在os cache内存缓存里面，没实际进入到磁盘文件，此时宕机，数据丢失。

  ![redolog策略2](../mysqldoc/images/redolog策略2.PNG)