一、左连接

select * from A left join B on A.key = B.key 左连接

示意图：

![](https://github.com/heartccace/mysql/tree/master/images/左连接.jpg)

### 二、 右连接 

示意图：

select * from A right join B on A.key = B.key 右连接

![](https://github.com/heartccace/mysql/tree/master/images/右连接.jpg)

### 三、全连接

select * from A inner join B on A.key = B.key

![](https://github.com/heartccace/mysql/tree/master/images/内连接.jpg)

### 四、左连接去除B空

select * from A left join B on A.key = B.key where B.key is null

![](https://github.com/heartccace/mysql/tree/master/images/左连接去除B空.jpg)

### 五、右连接去除A空

select * from A right join B on A.key =B.key where A.key is null

![](https://github.com/heartccace/mysql/tree/master/images/右连接去除A空.jpg)

### 六、全连接

select * from A full outer join B on A.key = B.key（由于不支持外连接可以使用 select * from A left join B on a.key =b.key union select * from A right join B on a.key,union关键字可以实现去重功能）

![](https://github.com/heartccace/mysql/tree/master/images/全连接.jpg)

### 七、全连接除AB合集

select * from A full outer join B on A.key = B.key where A.key is null  or B.key is null;

![](https://github.com/heartccace/mysql/tree/master/images/全连接去除AB合.jpg)