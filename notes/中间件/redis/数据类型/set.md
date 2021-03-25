## 集合 sort
- REDIS_SET （集合）是 SADD 、 SRANDMEMBER 等命令的操作对象， 它使用 REDIS_ENCODING_INTSET 和 REDIS_ENCODING_HT 两种方式编码

### 编码选择
- 第一个添加到集合的元素， 决定了创建集合时所使用的编码：

  - 如果第一个元素可以表示为 long long 类型值（也即是，它是一个整数）， 那么集合的初始编码为 REDIS_ENCODING_INTSET 。
  - 否则，集合的初始编码为 REDIS_ENCODING_HT 。

### 编码切换
- 如果一个集合使用 REDIS_ENCODING_INTSET 编码， 那么当以下任何一个条件被满足时， 这个集合会被转换成 REDIS_ENCODING_HT 编码：

  - intset 保存的整数值个数超过 server.set_max_intset_entries （默认值为 512 ）。
  - 试图往集合里添加一个新元素，并且这个元素不能被表示为 long long 类型（也即是，它不是一个整数）。

### 字典编码集合
当使用 REDIS_ENCODING_HT 编码时， 集合将元素保存到字典的键里面， 而字典的值则统一设为 NULL 。

### 求交集

``` 
# coding: utf-8

def sinter(*multi_set):

    # 根据集合的基数进行排序
    sorted_multi_set = sorted(multi_set, lambda x, y: len(x) - len(y))

    # 使用基数最小的集合作为基础结果集，有助于降低常数项
    result = sorted_multi_set[0].copy()

    # 剔除所有在 sorted_multi_set[0] 中存在
    # 但在其他某个集合中不存在的元素
    for elem in sorted_multi_set[0]:

        for s in sorted_multi_set[1:]:

            if (not elem in s):
                result.remove(elem)
                break

    return result
```
算法的复杂度为 O(N2) ， 执行步数为 S∗T ， 其中 S 为输入集合中基数最小的集合， 而 T 则为输入集合的数量。

### 并集
SUNION 和 SUNIONSTORE 两个命令所使用的求并集算法可以用 Python 表示如下：

``` 
# coding: utf-8

def sunion(*multi_set):

    result = set()

    for s in multi_set:
        for elem in s:
            # 重复的元素会被自动忽略
            result.add(elem)

    return result
```
算法的复杂度为 O(N) 。

### 差集
Redis 为 SDIFF 和 SDIFFSTORE 两个命令准备了两种求集合差的算法。

``` 
# coding: utf-8

def sdiff_1(*multi_set):

    result = multi_set[0].copy()

    sorted_multi_set = sorted(multi_set[1:], lambda x, y: len(x) - len(y))

    # 当 elem 存在于除 multi_set[0] 之外的集合时
    # 将 elem 从 result 中删除
    for elem in multi_set[0]:

        for s in sorted_multi_set:

            if elem in s:
                result.remove(elem)
                break

    return result
```


