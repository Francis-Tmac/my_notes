### 登录
``` 
[user_00@VM-77-41-centos ~]$ mysql -h 192.168.0.1 -u p2p_dev -p -P 3306
-h:host 远程主机
-u:username 用户名
-p:password 密码
-P:port 端口
mysql -u root -p
```

### 索引
- 查看表索引
``` 
mysql> show index from users;
+-------+------------+-----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name  | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+-----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| users |          0 | PRIMARY   |            1 | id          | A         |           6 |     NULL | NULL   |      | BTREE      |         |               |
| users |          1 | last_name |            1 | last_name   | A         |           6 |     NULL | NULL   |      | BTREE      |         |               |
| users |          1 | age       |            1 | age         | A         |           6 |     NULL | NULL   | YES  | BTREE      |         |               |
+-------+------------+-----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
3 rows in set (0.00 sec)

MySQL [work_hub]> show index from t_example;
+--------+------------+------------------------------+--------------+-------------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table  | Non_unique | Key_name                     | Seq_in_index | Column_name       | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+--------+------------+------------------------------+--------------+-------------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| t_lend |          0 | PRIMARY                      |            1 | FuiId             | A         |         666 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          0 | uniq_FuiBizId                |            1 | FuiBizId          | A         |         656 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          0 | uniq_FstrExternalId          |            1 | FstrExternalId    | A         |         656 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FstrOrderId              |            1 | FstrOrderId       | A         |         304 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FuiStatus_FuiIsProc      |            1 | FuiStatus         | A         |           4 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FuiStatus_FuiIsProc      |            2 | FuiIsProc         | A         |           6 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_update_time              |            1 | FuiUpdateTime     | A         |         379 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FuiBorrowerUid           |            1 | FuiBorrowerUid    | A         |         186 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FstrExternalBizId        |            1 | FstrExternalBizId | A         |         495 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FuiChannel_FuiCreateTime |            1 | FuiChannel        | A         |          21 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FuiChannel_FuiCreateTime |            2 | FuiCreateTime     | A         |         302 |     NULL | NULL   |      | BTREE      |         |               |
| t_lend |          1 | idx_FuiLenderUid             |            1 | FuiLenderUid      | A         |          16 |     NULL | NULL   |      | BTREE      |         |               |
+--------+------------+------------------------------+--------------+-------------------+-----------+-------------+----------+--------+------+------------+---------+---------------+

```

### 存储过程
```sql
// 定义分隔符为 ';;' 号, 默认情况下分隔符为 ';'
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000)do
    insert into t values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();

```

### 慢查询日志
``` sql
mysql> show variables like 'slow_query%';
+---------------------+-----------------------------------------+
| Variable_name       | Value                                   |
+---------------------+-----------------------------------------+
| slow_query_log      | ON                                      |
| slow_query_log_file | /usr/local/var/mysql/XYSZSX066-slow.log |
+---------------------+-----------------------------------------+
2 rows in set (0.00 sec)

mysql> show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 2.000000 |
+-----------------+----------+
1 row in set (0.00 sec)
```

``` sql
# Time: 2021-04-23T02:53:44.566996Z
# User@Host: skip-grants user[root] @ localhost []  Id:    15
# Query_time: 0.013812  Lock_time: 0.000137 Rows_sent: 10001  Rows_examined: 10001
SET timestamp=1619146424;
select * from t_test_3 where a between 10000 and 20000;
```
***

### 前缀索引
直接创建完整索引，这样可能比较占用空间；
- 查看列上不同的值
```sql
mysql> select * from users;
+----+-----------+------------+------+
| id | last_name | first_name | age  |
+----+-----------+------------+------+
|  1 | tom       | hiddleston |   30 |
|  2 | donald    | trump      |   81 |
|  3 | morgan    | freeman    |   40 |
|  4 | stark     | tony       |   21 |
|  5 | jeff      | dean       |   50 |
|  6 | kakaxi    | dear       |   35 |
+----+-----------+------------+------+

mysql> select count(distinct left(first_name,3)) as L from users;
+---+
| L |
+---+
| 5 |
+---+

mysql> select
    -> count(distinct left(first_name,4)) as L1,
    -> count(distinct left(first_name,3)) as L2,
    -> count(distinct left(first_name,5)) as L3
    -> from users;
+----+----+----+
| L1 | L2 | L3 |
+----+----+----+
|  6 |  5 |  6 |
+----+----+----+
1 row in set (0.00 sec)

```
- 使用前缀索引，定义好长度，就可以做到既节省空间，又不用额外增加太多的查询成本。
- 建立索引时关注的是区分度，区分度越高越好。因为区分度越高，意味着重复的键值越少。
- 使用前缀索引就用不上覆盖索引对查询性能的优化了，选择是否使用前缀索引时需要考虑的一个因素。

#### 其他方式
- 倒序存储
- 使用hash 字段，再创建一个整数字段，来保存身份证的校验码，同时在这个字段上创建索引。
- 倒序和 hash 都不支持范围查询

*** 
### flush
- 当内存数据页跟磁盘数据页内容不一致的时候，这个内存页为“脏页”。内存数据写入到磁盘后，内存和磁盘上的数据页的内容就一致了，称为“干净页”。
- 什么场景会引发数据库 flush
    - InnoDB 的 redo log 写满了。这时候系统会停止所有更新操作，把 checkpoint 往前推进，redo log 留出空间可以继续写。
    - 系统内存不足。当需要新的内存页，而内存不够用的时候，就要淘汰一些数据页，空出内存给别的数据页使用。如果淘汰的是“脏页”，就要先将脏页写到磁盘。
    - MySQL 认为系统“空闲”的时候
    - MySQL 正常关闭的情况。这时候，MySQL 会把内存的脏页都 flush 到磁盘上

```sql
mysql> show variables like  'innodb_io_capacity%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_io_capacity     | 200   |
| innodb_io_capacity_max | 2000  |
+------------------------+-------+
2 rows in set (0.01 sec)
```

### 表存储
innodb\_file\_per_table 控制的：
-  这个参数设置为 OFF 表示的是，表的数据放在系统共享表空间，也就是跟数据字典放在一起；
-  这个参数设置为 ON 表示的是，每个 InnoDB 表数据存储在一个以 .ibd 为后缀的文件中。
- 一个表单独存储为一个文件更容易管理，而且在你不需要这个表的时候，通过 drop table 命令，系统就会直接删除这个文件。而如果是放在共享表空间中，即使表删掉了，空间也是不会回收的。
