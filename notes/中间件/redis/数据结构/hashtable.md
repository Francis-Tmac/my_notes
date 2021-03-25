## hashtable 字典使用

dict 结构体：
``` 
/* 字典的主操作类，对dictht结构再次包装  */
typedef struct dict {
    // 字典类型
    dictType *type;
    // 私有数据
    void *privdata;
    // 一个字典中有两个哈希表
    dictht ht[2];
    // 数据动态迁移的下标位置
    long rehashidx; 
    // 当前正在使用的迭代器的数量
    int iterators; 
} dict;

```

dictht结构体:
```
/* 哈希表结构 */
typedef struct dictht {
    // 散列数组。
    dictEntry **table;
    // 散列数组的长度
    unsigned long size;
    // sizemask等于size减1
    unsigned long sizemask;
    // 散列数组中已经被使用的节点数量
    unsigned long used;
} dictht;

```

dictType结构体:
```
/* 定义了字典操作的公共方法，类似于adlist.h文件中list的定义，将对节点的公共操作方法统一定义。搞不明白为什么要命名为dictType */
typedef struct dictType {
    /* hash方法，根据关键字计算哈希值 */
    unsigned int (*hashFunction)(const void *key);
    /* 复制key */
    void *(*keyDup)(void *privdata, const void *key);
    /* 复制value */
    void *(*valDup)(void *privdata, const void *obj);
    /* 关键字比较方法 */
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    /* 销毁key */
    void (*keyDestructor)(void *privdata, void *key);
    /* 销毁value */
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

```

dictEntry结构体:
```
/* 保存键值（key - value）对的结构体，类似于STL的pair。*/
typedef struct dictEntry {
    // 关键字key定义
    void *key;  
    // 值value定义，只能存放一个被选中的成员
    union {
        void *val;      
        uint64_t u64;   
        int64_t s64;    
        double d;       
    } v;
    // 指向下一个键值对节点
    struct dictEntry *next;
} dictEntry;

```

四个结构体之间关系:
![hashtable](../../../../img/middleware/redis-hashtable.jpg)

#### 哈希算法

#### 解决键冲突
链地址法，程序总是将新节点添加到链表的表头节点位置（复杂度为O(1)）,排在其他已有节点的前面。

#### rehash
1. 为字典的ht[1] 哈希表分配空间，这个哈希表的空间大小取决于要执行的操作，以及ht[0]当前包含的键值对数量：
2. 如果是扩展操作，那么ht[1] 的大小为ht[0] 的一倍
3. 如果是收缩操作，那么ht[1] 的大小为ht[0] 的一半
4. 将保存在ht[0] 中的所有键值对rehash 到 ht[1] 中，rehash 是重新计算键的哈希值和索引值，然后将键值对放到ht[1]哈希表的指定位置上。
5. 当ht[0] 包含的所有键值对都迁移到ht[1] 之后，释放ht[0],将ht[1]设置为ht[0],并在ht[1]新创建一个空白的哈希表，为下一次rehash 准备。

#### 扩展与收缩
服务器没有执行 BGSAVE 命令或者 BGREWRITEAOF, 并且负载因子大于等于1 
服务器正在执行 BGSAVE 命令或者 BGREWRITEAOF, 并且负载因子大于等于5

负载因子 = 哈希表已保存节点数量/哈希表大小

#### 渐进式hash 
1. 为ht[1] 分配空间，让字典同时持有ht[0] 和 ht[1] 两个哈希表
2. 在字典中维持一个索引计数器变量 rehashindx,并将他的值设置为0，表示rehash 正式开始。
3. 在rehash 进行期间，每次对字典执行添加，删除，查找或者更新操作时，程序除了执行指定的操作外，还会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到htp[1],当rehash工作完成之后，程序将rehashidx属性增加一。
4. 随着字典的操作不断执行，最终ht[0]的所有键值对都会被rehash到ht[1]，这时将 rehashidx 设置为-1，表示rehash操作完成。

##### 哈希表操作
字典的删除，查找，更新邓操作会在两个哈希表上。添加到字典的键值一律会被保存到ht[1]里面。
