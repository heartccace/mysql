MySQL官方文档地址（https://dev.mysql.com/doc/refman/5.7/en/explain.html）

一、explain + sql执行后显示内容

![](https://github.com/heartccace/mysql/blob/master/images/exlpain包含信息.png)

select 查询的序列号包含一组数字，表示查询中执行select子句或操作表的顺序，包含三种情况：

1. id相同执行顺序由上至下

   ![](https://github.com/heartccace/mysql/blob/master/images/explain之id相同.jpg)

2. id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

   ![](https://github.com/heartccace/mysql/blob/master/images/explain之id不同.jpg)

3. id值不同同时存在，值越高越先被执行，相同从上至下一次执行(derived2 代表衍生表，后面的数字代表id，是某个id生成的衍生表) 

   ![](https://github.com/heartccace/mysql/blob/master/images/explain之id相同又不同.jpg)

二、explain + sql之select_type

1. select_type取值

   ![](https://github.com/heartccace/mysql/blob/master/images/explain之select_type.jpg)

   SIMPLE -  简单的select查询，查询中不包含子查询或者UNION

   PRIMARY - 查询中包含任何复杂的子部分，最外层查询则被标记为PRIMARY

   SUBQUERY - 在select或where中包含子查询

   DERIVED - 在FROM列表中包含子查询被标记为DERIVE（衍生）MYSQL会递归执行这些子查询，把结果放到临时表中

   UNION - 若第二个select出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层select将被标记为DERIVED

   UNION RESULT: 从UNION表获取结果的select

2. 作用:查询类型，主要用于区分普通查询、联合查询、子查询等复杂查询

​    