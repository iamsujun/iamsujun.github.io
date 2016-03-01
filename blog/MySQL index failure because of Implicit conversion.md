# MySQL隐式转换导致索引失效
> 隐式转换，导致索引失效，在日常开发很容易被忽略掉。数据量不大时，是否使用索引效影响不大。

数据表里有200万的数据，`sNum(varchar)`字段建有索引，基于sNum进行查询，`select * from table1 where sNum = 33521`，速度奇慢无比。explain一下，发现进行了全表查询，查了下原因，原来是进行了隐式转换，索引失效。

调整sql，`select * from table1 where sNum = "33521"`，使用了索引，不再慢了。

参考：
- [Mysql Type Conversion in Expression Evaluation](http://dev.mysql.com/doc/refman/5.7/en/type-conversion.html)
- [MySQL int转换成varchar引发的慢查询](http://www.cnblogs.com/billyxp/archive/2013/05/31/3110016.html)
