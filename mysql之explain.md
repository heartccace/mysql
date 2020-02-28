MySQL官方文档地址（https://dev.mysql.com/doc/refman/5.7/en/explain.html）
https://dev.mysql.com/doc/refman/5.7/en/explain-output.html

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

三、explain之type（https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-join-types）

1. 取值情况

   ![](https://github.com/heartccace/mysql/blob/master/images/explain之type.jpg)

   查询使用何种类型，从最好到最差

   system > const > eq_ref > ref > rang  > index > all(常用)

   system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > rang  > index > all(所有)

   优化至少得到rang，最好达到ref

2. 取值详解

   -  system: 表只有一行记录（等于系统表），这是const类型的特列，平时不会吃先，这个也可以忽略不记
   - const：表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快，如将主键位于where列表中（如where id =10），Mysql就能将该查询转换为一个常量；
   - eq_ref：唯一性索引，对于每个索引键，表中只有一条记录与之匹配。常见与主键或唯一索引扫描
   - ref：非唯一性索引扫描，返回匹配单独值的所有行，本质上也是一种索引访问，他返回所有匹配某个单独值的行，然而他可能会找到很多符合条件的行，所以它属于查找和扫描的混合体
   - range：只检索给定范围的行，使用一个索引来选择行。key列显示使用哪个索引，一般就是在你的where语句中初爱西安between、<、>、in等查询，这种范围扫描索引比全表要好，因为它只需要开始于索引的某一点，而结束与另一点，不用扫描全部索引。
   - index：index与all的区别为index'里欸选哪个只遍历索引树。通常逼all快，因为索引文件通常比数据文件小（也就是说虽然all'和index都是读全表，但index是从索引中读取，all是从硬盘中读取）

四、explain之possible_key / key

1. possible_key：显示可能运用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，<font color="red">但不一定被查询实际使用</font>
2. key：实际使用的索引。如果为NULL，则没有使用索引，若查询语句使用了覆盖索引，则该索引仅出现在key列表中
3. key_len：显示的职位索引字段的最大可能长度，并非实际长度，即key_len是根据表定义计算得到，不是通过表内检索出。
4. ref：显示索引的那一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。
5. rows：根据表统计信息及索引选用情况，大致估算除找到所需的记录所需读取的行数。

五、explain值extra额外信息

1. 取值情况

   - Using Sort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，mysql中无法利用索引完成的排序操作称为“文件排序”（性能差）
     案例：

     ​			![](https://github.com/heartccace/mysql/blob/master/images/explain之extra.jpg)

   - Using temporary：使用临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by 和分组查询group by(创建临时表，性能最差)

     案列：

     ​	![](https://github.com/heartccace/mysql/blob/master/images/explain之useing template.jpg)

   - Using index：表示相应的select操作中使用了覆盖索引，避免访问了表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找。如果没有出现using where，表明索引用来读取数据而非执行查找动作。

     覆盖索引（索引覆盖）：就是select的数据列只用从索引中就可以取得，不必读取数据行，mysql可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，换句话说查询列要被所建索引覆盖。

     **注意：如果要使用覆盖索引一定要注意select列表中只取出所需的列。不可使用select *，否则会出现回表**

2. 案例分析

   案例：
   			![](https://github.com/heartccace/mysql/blob/master/images/explain案例分析.jpg)

   执行顺序：首先查询t2表—> t1 (查询t1结果形成一张衍生表derived3) —> t3 —> derived3 —> union

### 六、索引优化

1. 单表情况

   - 使用联合索引时，mysql采用BTree来存储索引。进行查询排序时是按照索引的先后顺序进行排序，当其中索引出现相同值的时候采用另外的索引。此时如果where后跟的条件被加入索引，且使用范围查询（in，<,>）等会导致索引失效，导致查询性能降低。
     案例：

     ![](https://github.com/heartccace/mysql/blob/master/images/单表索引未优化.jpg)

     ![](https://github.com/heartccace/mysql/blob/master/images/单表未优化结果.jpg)

     本案例采用联合主键，根据category_id、comments、views三个字段组成一个idx_article_ccv主键，通过sql（select id from article where category_id=1 and comments >1 order by views desc limit 1）查询，导致排序未使用到主键而采用外部索引排序，导致性能下降。

     原因分析：其中comments > 1这个条件会使索引失效导致

     解决方案：索引元素去掉comments，只根据categor_id和views创建索引。

     解决后：

     ![](https://github.com/heartccace/mysql/blob/master/images/单表优化索引.jpg)

     ![](https://github.com/heartccace/mysql/blob/master/images/单表优化结果.jpg)

2. 两表情况

   当两个表进行关联查询采用left/right join时，可根据关联字段进行优化，优化原则时

   - 当采用left join时，优化方案时在右表的关联字段上建立索引。
   - 当采用right join时，优化方案实在左表关联字段上建立索引

七、索引失效

1. 全值匹配

2. 最佳左前缀法则

   ![](https://github.com/heartccace/mysql/blob/master/images/复合索引失效.jpg)

   **如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列**否则会导致索引失效

3. 不在索引列上做任何操作（计算、函数、（自动or手动类型转化）），会导致索引失效而全表扫描

4. 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *

5. 存储引擎不能使用索引中范围条件右边的列（范围右边全失效）

6. mysql在使用不等于（!=,<>）的时候无法索引导致全表扫描

7. is null， is not null也无法使用索引

8. like以通配符开头（‘%abc’）mysql索引失效导致全表扫描

   ‘%...%’导致索引失效，可以通过覆盖索引解决问题。前提查询字段必须与覆盖索引一致（除在自增id外）

9. 字符串不加单引号导致索引失效

10. 少用or，用它来连接时索引失效

### 八、总结

1. 慢查询的开启并捕获
2. explain + 慢sql分析
3. show profile查询SQL在mysql服务器里面的执行细节和生命周期
4. SQL数据库服务器的参数调优
5. exists和in的区别
   - ​	SELECT * FROM A where exists(select  1 from B where B.id=A.id)当A的数据集小于B的数据集时exists优于in

### 九、order by排序优化

1. ​	order by 子句，尽量使用index方式排序，避免使用filesort

2. 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

3. 如果不在索引列上，filesort有两种算法

4. mysql就要启动双路排序和单路排序

   总结：

   - mysql两种排序：文件排序或者扫码有序索引排序

   - mysql能为排序与查询使用相同的索引

     KEY a_b_c(a,b,c)

     — order by a

     — order by a，b

     — order by a,b,c

     — order by a DESC, b desc, c desc

   - 如果where使用索引的最左前缀定义为常量，则order by能使用索引

     — where  a = const ORDER BY b,c

     — where  a = const and b=const ORDER BY c

     — where  a = const and b > const ORDER BY b,c

     不能使用索引进行排序
     —  order by a asc, b desc, c desc /* 排序不一致 */

     — where g=const order by  b ,c /* 丢失索引a */

     — where a=const order by  c /* 丢失索引b */

     — where a in (....) order by  b ,c /* 杜宇排序来说，多个相等条件也是范围查询*/

### 十、慢查询

1. 开启慢查询
   - show variables like "%slow_query_log%"
   - set globle slow_query_log=1(只能当前数据库生效)
   - 永久生效（在my.ini中配置[sysqld]下增加 show_query_log =1 show_query_log_file="保存位置"）
2. 查看慢sql记录时间（show variables like 'long_query_time'）（set gloal long_query_time=3）设置慢查询记录sql时间为3s
