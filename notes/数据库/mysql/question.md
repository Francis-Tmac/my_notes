### 一条查询语句的执行过程
客户端通过连接器将SQL 传到MySQL ，MySQL server 中先进行语法分析和词法分析，再使用优化器优化SQL ，然后查看是否开启缓存如果开启在缓存中查看是否存在。若没有则到存储引擎中查找数据，在执行引擎中会现在缓存页中查找

order by 的字段，如果不是索引


1、日志文件undo log和redo log用途以及两者区别。
2、innodb buffer pool，lru list, flush list,innodb_old_block_time,innodb_old_block_pct。
3、mysql如何实现acid。
4、mysql数据页的格式(一页数据能存多少记录？一行记录太长怎么存储)，几种数据行格式(compact、dynamic)B+树的插入和删除，只需要知道大概格式。
5、事务的隔离级别和各种锁。
6、什么字段适合建索引，什么不适合，建索引有哪些基本原则。
7、覆盖索引，几种优化mrr，icp优化需要理解。
8、binlog的格式，gtid和xid是什么东西。
9、主从同步原理，半同步，全同步，异步。如何进行主从切换。
10、mysql几种高可用架构，组复制。
11、业界的一些分布式数据可以大概了解下，阿里云drds、polardb、mariadb。

分库分表的场景

db容灾、mysql数据容灾

- 事务隔离
- 数据库性能优化

- mysql运行情况查看，对应指标，命令
- 如何判断索引是好的索引
- explain具体怎么分析，关注重点
- 事务隔离
- 读已提交和可重复读分别是怎么实现的
- 间隙锁是怎么产生的
- mysql性能调优
- 覆盖索引



- 给定user表包含id，name字段，id有主键索引，name有普通索引，一共有10亿条记录，给定硬件CPU 1核，内存1G，硬盘20T。执行:select * from user where name = 'xx' limit 100000,10
        - 说一下所有能想到的知识点
        - limit 100万,10 和1000万，10的区别

db 隔离机制  MVCC 聚簇索引  间隙锁  next-key-locks 这些锁怎么产生


1.  sql问题，树形结构，查出子节点大于3的节点
关于group by查不到数据的时候不会返回，需要在外层在包一层left/inner join，并使用ifnull(num,0)来做为空判断

12. mysql和redis区别？