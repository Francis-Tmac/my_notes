因为 ziplist 节约内存的性质， 哈希键、列表键和有序集合键初始化的底层实现皆采用 ziplist

Ziplist 是由一系列特殊编码的内存块构成的列表， 一个 ziplist 可以包含多个节点（entry）， 每个节点可以保存一个长度受限的字符数组（不以 \0 结尾的 char 数组）或者整数， 包括：

- 字符数组
  - 长度小于等于 63 （26−1）字节的字符数组
  - 长度小于等于 16383 （214−1） 字节的字符数组
  - 长度小于等于 4294967295 （232−1）字节的字符数组
- 整数
  - 4 位长，介于 0 至 12 之间的无符号整数
  - 1 字节长，有符号整数
  - 3 字节长，有符号整数
  - int16_t 类型整数
  - int32_t 类型整数
  - int64_t 类型整数
  
#### ziplist 构成
``` 
area        |<---- ziplist header ---->|<----------- entries ------------->|<-end->|

size          4 bytes  4 bytes  2 bytes    ?        ?        ?        ?     1 byte
            +---------+--------+-------+--------+--------+--------+--------+-------+
component   | zlbytes | zltail | zllen | entry1 | entry2 |  ...   | entryN | zlend |
            +---------+--------+-------+--------+--------+--------+--------+-------+
                                       ^                          ^        ^
address                                |                          |        |
                                ZIPLIST_ENTRY_HEAD                |   ZIPLIST_ENTRY_END
                                                                  |
                                                         ZIPLIST_ENTRY_TAIL
```  
各域的作用如下

域	 | 长度/类型	| 域的值
---| ---| ---
zlbytes	| uint32_t	| 整个 ziplist 占用的内存字节数，对 ziplist 进行内存重分配，或者计算末端时使用。
zltail	| uint32_t	| 到达 ziplist 表尾节点的偏移量。 通过这个偏移量，可以在不遍历整个 ziplist 的前提下，弹出表尾节点。
zllen	| uint16_t	| ziplist 中节点的数量。 当这个值小于 UINT16_MAX （65535）时，这个值就是 ziplist 中节点的数量； 当这个值等于 UINT16_MAX 时，节点的数量需要遍历整个 ziplist 才能计算得出。
entryX	| ?	| ziplist 所保存的节点，各个节点的长度根据内容而定。
zlend	| uint8_t	| 255 的二进制值 1111 1111 （UINT8_MAX） ，用于标记 ziplist 的末端。


以下是用于操作 ziplist 的函数：

函数名 | 	作用 | 	算法复杂度
---| ---| ---
ziplistNew | 	创建一个新的 ziplist | 	θ(1)
ziplistResize	 | 重新调整 ziplist 的内存大小 | 	O(N)
ziplistPush | 	将一个包含给定值的新节点推入 ziplist 的表头或者表尾	 | O(N2)
zipEntry	 | 取出给定地址上的节点，并将它的属性保存到 zlentry 结构然后返回 | 	θ(1)
ziplistInsert	 | 将一个包含给定值的新节点插入到给定地址	 | O(N2)
ziplistDelete	 | 删除给定地址上的节点	 | O(N2)
ziplistDeleteRange | 	在给定索引上，连续进行多次删除	 | O(N2)
ziplistFind	 | 在 ziplist 中查找并返回包含给定值的节点	 | O(N)
ziplistLen	 | 返回 ziplist 保存的节点数量	 | O(N)
ziplistBlobLen | 	以字节为单位，返回 ziplist 占用的内存大小 | 	θ(1)
  
#### 节点构成
一个 ziplist 可以包含多个节点，每个节点可以划分为以下几个部分：

``` 
area        |<------------------- entry -------------------->|

            +------------------+----------+--------+---------+
component   | pre_entry_length | encoding | length | content |
            +------------------+----------+--------+---------+
```

##### pre_entry_length
pre_entry_length 记录了前一个节点的长度，通过这个值，可以进行指针计算，从而跳转到上一个节点。

``` 
area        |<---- previous entry --->|<--------------- current entry ---------------->|

size          5 bytes                   1 byte             ?          ?        ?
            +-------------------------+-----------------------------+--------+---------+
component   | ...                     | pre_entry_length | encoding | length | content |
            |                         |                  |          |        |         |
value       |                         | 0000 0101        |    ?     |   ?    |    ?    |
            +-------------------------+-----------------------------+--------+---------+
            ^                         ^
address     |                         |
            p = e - 5                 e
```

上图展示了如何通过一个节点向前跳转到另一个节点： 用指向当前节点的指针 e ， 减去 pre_entry_length 的值（0000 0101 的十进制值， 5）， 得出的结果就是指向前一个节点的地址 p 。

- 根据编码方式的不同， pre_entry_length 域可能占用 1 字节或者 5 字节：

  - 1 字节：如果前一节点的长度小于 254 字节，便使用一个字节保存它的值。
  - 5 字节：如果前一节点的长度大于等于 254 字节，那么将第 1 个字节的值设为 254 ，然后用接下来的 4 个字节保存实际长度。

#### encoding 和 length
encoding 和 length 两部分一起决定了 content 部分所保存的数据的类型（以及长度）。

其中， encoding 域的长度为两个 bit ， 它的值可以是 00 、 01 、 10 和 11 ：

- 00 、 01 和 10 表示 content 部分保存着字符数组。
  - 11 表示 content 部分保存着整数。
  - 以 00 、 01 和 10 开头的字符数组的编码方式如下：

#### content
以下是一个保存着字符数组 hello world 的节点的例子：
``` 
area      |<---------------------- entry ----------------------->|

size        ?                  2 bit      6 bit    11 byte
          +------------------+----------+--------+---------------+
component | pre_entry_length | encoding | length | content       |
          |                  |          |        |               |
value     | ?                |    00    | 001011 | hello world   |
          +------------------+----------+--------+---------------+
```
encoding 域的值 00 表示节点保存着一个长度小于等于 63 字节的字符数组， length 域给出了这个字符数组的准确长度 —— 11 字节（的二进制 001011）， content 则保存着字符数组值 hello world 本身（为了方便表示， content 部分使用字符而不是二进制表示）。

以下是另一个节点，它保存着整数 10086 ：

``` 
area      |<---------------------- entry ----------------------->|

size        ?                  2 bit      6 bit    2 bytes
          +------------------+----------+--------+---------------+
component | pre_entry_length | encoding | length | content       |
          |                  |          |        |               |
value     | ?                |    11    | 000000 | 10086         |
          +------------------+----------+--------+---------------+
```
encoding 域的值 11 表示节点保存的是一个整数； 而 length 域的值 000000 表示这个节点的值的类型为 int16_t ； 最后， content 保存着整数值 10086 本身（为了方便表示， content 部分用十进制而不是二进制表示）。

#### 创建新 ziplist
``` 
area        |<---- ziplist header ---->|<-- end -->|

size          4 bytes   4 bytes 2 bytes  1 byte
            +---------+--------+-------+-----------+
component   | zlbytes | zltail | zllen | zlend     |
            |         |        |       |           |
value       |  1011   |  1010  |   0   | 1111 1111 |
            +---------+--------+-------+-----------+
                                       ^
                                       |
                               ZIPLIST_ENTRY_HEAD
                                       &
address                        ZIPLIST_ENTRY_TAIL
                                       &
                               ZIPLIST_ENTRY_END
```
#### 添加

添加和删除 ziplist 节点有可能会引起连锁更新，因此，添加和删除操作的最坏复杂度为 O(N2) ，不过，因为连锁更新的出现概率并不高，所以一般可以将添加和删除操作的复杂度视为 O(N) 。



#### 连锁更新可能会带来多个节点的空间重分配操作。
