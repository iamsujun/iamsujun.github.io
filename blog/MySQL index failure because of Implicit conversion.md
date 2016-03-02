# Mysql隐式转换导致索引失效
> 隐式转换，导致索引失效，在日常开发很容易被忽略掉。数据量不大时，是否使用索引效影响不大。

数据表里有200万的数据，`sNum(varchar)`字段建有索引，基于sNum进行查询，`select * from iamsujun where sNum = 5007`，速度奇慢无比。

```
CREATE TABLE `iamsujun` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `sNum` varchar(255) NOT NULL COMMENT 'number',
  `sExt` varchar(255) NOT NULL COMMENT 'extra',
  PRIMARY KEY (`id`),
  KEY `sNum` (`sNum`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 COMMENT='mysql implicit conversion test table';
```

Explain一下，发现进行了全表查询，索引失效。

```
jun@test 10:00:58>EXPLAIN SELECT * FROM iamsujun WHERE sNum = 5007;
+----+-------------+----------+------+---------------+------+---------+------+---------+-------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows    | Extra       |
+----+-------------+----------+------+---------------+------+---------+------+---------+-------------+
|  1 | SIMPLE      | iamsujun | ALL  | sNum          | NULL | NULL    | NULL | 2033521 | Using where |
+----+-------------+----------+------+---------------+------+---------+------+---------+-------------+
1 row in set (0.00 sec)
```

查了下原因，原来是进行了隐式转换，隐式转换示例：

```
jun@test 10:09:06>SELECT "a"=1;
+-------+
| "a"=1 |
+-------+
|     0 |
+-------+
1 row in set, 1 warning (0.00 sec)

jun@test 10:09:29>SHOW WARNINGS;
+---------+------+---------------------------------------+
| Level   | Code | Message                               |
+---------+------+---------------------------------------+
| Warning | 1292 | Truncated incorrect DOUBLE value: 'a' |
+---------+------+---------------------------------------+
1 row in set (0.00 sec)
```

加上引号，避免隐式转换示例：

```
jun@test 10:10:29>SELECT "a"="1";
+---------+
| "a"="1" |
+---------+
|       0 |
+---------+
1 row in set (0.00 sec)

jun@test 10:10:39>SHOW WARNINGS;
Empty set (0.00 sec)
```

调整SQL加上引号，`select * from iamsujun where sNum = "5007"`，查询使用了索引，不再慢了。

```
jun@test 10:00:56>EXPLAIN SELECT * FROM iamsujun WHERE sNum = "5007";
+----+-------------+----------+------+---------------+------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | iamsujun | ref  | sNum          | sNum | 767     | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+------+---------+-------+------+-----------------------+
1 row in set (0.00 sec)
```

----

参考：
- [Mysql Type Conversion in Expression Evaluation](http://dev.mysql.com/doc/refman/5.7/en/type-conversion.html)
- [MySQL int转换成varchar引发的慢查询](http://www.cnblogs.com/billyxp/archive/2013/05/31/3110016.html)
