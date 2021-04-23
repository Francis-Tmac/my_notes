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

MySQL [fund_hub]> show index from t_example;
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

