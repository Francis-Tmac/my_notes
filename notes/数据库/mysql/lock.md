## 全局锁

## 表锁
### lock tables … read/write

### MDL（metadata lock)

## 行锁

- 在 InnoDB 事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放。这个就是两阶段锁协议。

### 死锁和死锁检测
