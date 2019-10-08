### 一、左连接

select * from A left join B on A.key = B.key 左连接

示意图：

![][左连接.jpg](https://github.com/heartccace/mysql/blob/master/images/左连接.jpg)

### 二、 右连接 

select * from A right join B on A.key = B.key 右连接

![][右连接.jpg](https://github.com/heartccace/mysql/blob/master/images/右连接.jpg)

### 三、全连接

select * from A inner join B on A.key = B.key

![][内连接.jpg](https://github.com/heartccace/mysql/blob/master/images/内连接.jpg)

### 四、左连接去除B空

select * from A left join B on A.key = B.key where B.key is null

![][左连接去除B空.jpg](https://github.com/heartccace/mysql/blob/master/images/左连接去除B空.jpg)

### 五、右连接去除A空

select * from A right join B on A.key =B.key where A.key is null

![][右连接去除A空.jpg](https://github.com/heartccace/mysql/blob/master/images/右连接去除A空.jpg)

### 六、全连接

select * from A full outer join B on A.key = B.key

![][全连接.jpg](https://github.com/heartccace/mysql/blob/master/images/全连接.jpg)

### 七、全连接除AB合集

select * from A full outer join B on A.key = B.key where A.key is null  or B.key is null;

![][全连接去除AB合.jpg](https://github.com/heartccace/mysql/blob/master/images/全连接去除AB合.jpg)