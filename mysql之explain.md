一、explain + sql执行后显示内容

![](https://github.com/heartccace/mysql/blob/master/images/exlpain包含信息.png)

select 查询的序列号包含一组数字，表示查询中执行select子句或操作表的顺序，包含三种情况：

1. id相同执行顺序由上至下

   ![]

2. id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

3. id值不同同时存在，值越高越先被执行，相同从上至下一次执行