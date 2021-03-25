## 整数集合
#### 应用
Intset 是集合键的底层实现之一，如果一个集合：

只保存着整数元素；
元素的数量不多；

那么 Redis 就会使用 intset 来保存集合元素。

#### 数据结构
```$xslt
typedef struct intset {

    // 保存元素所使用的类型的长度
    uint32_t encoding;

    // 元素个数
    uint32_t length;

    // 保存元素的数组
    int8_t contents[];

} intset;
```
encoding 的值可以是以下三个常量之一（定义位于 intset.c ）：
```
#define INTSET_ENC_INT16 (sizeof(int16_t))
#define INTSET_ENC_INT32 (sizeof(int32_t))
#define INTSET_ENC_INT64 (sizeof(int64_t))
```

contents 数组是实际保存元素的地方，数组中的元素有以下两个特性：

- 元素不重复；
- 元素在数组中由小到大排列；

操作 | 	函数 | 	复杂度
--- | --- | ---
创建 intset	| intsetNew	| θ(1)
删除intset |	无	 |	无
添加新元素（不升级）|	intsetAdd |	O(N)
添加新元素（升级）|	intsetUpgradeAndAdd |	O(N)
按索引获取元素	| _intsetGet |	θ(1)
按索引设置元素	| _intsetSet |	θ(1)
查找元素，返回索引 |	intsetSearch	| O(lgN)
删除元素	| intsetRemove |	O(N)

#### 升级

intsetUpgradeAndAdd 需要完成以下几个任务：

- 对新元素进行检测，看保存这个新元素需要什么类型的编码；
- 将集合 encoding 属性的值设置为新编码类型，并根据新编码类型，对整个 contents 数组进行内存重分配。
- 调整 contents 数组内原有元素在内存中的排列方式，从旧编码调整为新编码。
- 将新元素添加到集合中。

整个过程中，最复杂的就是第三步，让我们用一个例子来理解这个步骤。

#### 升级实例
假设有一个 intset ，里面有三个用 int16_t 方式保存的数值，分别是 1 、 2 和 3 ，结构如下：

```
intset->encoding = INTSET_ENC_INT16;
intset->length = 3;
intset->contents = [1, 2, 3];
```
其中， intset->contents 在内存中的排列如下：

```$xslt
bit     0    15    31    47
value   |  1  |  2  |  3  |
```
现在，我们将一个长度为 int32_t 的值 65535 加入到集合中， intset 需要执行以下步骤：

将 encoding 属性设置为 INTSET_ENC_INT32 。

根据 encoding 属性的值，对 contents 数组进行内存重分配。

重分配完成之后， contents 在内存中的排列如下：

```$xslt
bit     0    15    31    47     63        95       127
value   |  1  |  2  |  3  |  ?  |    ?    |    ?    |
```
contents 数组现在共有可容纳 4 个 int32_t 值的空间。

因为原来的 3 个 int16_t 值还“挤在” contents 前面的 48 个位里， 所以程序需要移动它们并转换类型， 让它们适应集合的新编码方式。

首先是移动 3 ：

```$xslt
bit     0    15    31    47     63        95       127
value   |  1  |  2  |  3  |  ?  |    3    |    ?    |
                       |             ^
                       |             |
                       +-------------+
                     int16_t -> int32_t
```
接着移动 2 ：

```$xslt
bit     0    15    31   47     63        95       127
value   |  1  |  2  |    2     |    3    |    ?    |
                 |       ^
                 |       |
                 +-------+
            int16_t -> int32_t
```
最后，移动 1 ：

```$xslt
bit     0   15    31   47     63        95       127
value   |    1     |    2     |    3    |    ?    |
            | ^
            V |
    int16_t -> int32_t
```
最后，将新值 65535 添加到数组：

```$xslt
bit     0   15    31   47     63        95       127
value   |    1     |    2     |    3    |  65535  |
                                             ^
                                             |
                                            add
```
将 intset->length 设置为 4 。

至此，集合的升级和添加操作完成，现在的 intset 结构如下：

```$xslt
intset->encoding = INTSET_ENC_INT32;
intset->length = 4;
intset->contents = [1, 2, 3, 65535];
```
#### 关于升级
第一，从较短整数到较长整数的转换，并不会更改元素里面的值。
在 C 语言中，从长度较短的带符号整数到长度较长的带符号整数之间的转换（比如从 int16_t 转换为 int32_t ）总是可行的（不会溢出）、无损的。

另一方面，从较长整数到较短整数之间的转换，可能是有损的（比如从 int32_t 转换为 int16_t ）。

因为 intset 只进行从较短整数到较长整数的转换（也即是，只“升级”，不“降级”），因此，“升级”操作并不会修改元素原有的值

第二，集合编码元素的方式，由元素中长度最大的那个值来决定。
就像前面演示的例子一样， 当要将一个 int32_t 编码的新元素添加到集合时， 集合原有的所有 int16_t 编码的元素， 都必须转换为 int32_t 。

尽管这个集合真正需要用 int32_t 长度来保存的元素只有一个， 但整个集合的所有元素都必须转换为这种类型。

#### 其他操作
读取
有两种方式读取 intset 的元素，一种是 _intsetGet ，另一种是 intsetSearch ：

- _intsetGet 接受一个给定的索引 pos ，并根据 intset->encoding 的值进行指针运算，计算出给定索引在 intset->contents 数组上的值。
- intsetSearch 则使用二分查找算法，判断一个给定元素在 contents 数组上的索引。

写入

- 除了前面介绍过的 intsetAdd 和 intsetUpgradeAndAdd 之外， _intsetSet 也对集合进行写入操作： 它接受一个索引 pos ，以及一个 new_value ，将 contents 数组 pos 位置的值设为 new_value 。

删除
- 删除单个元素的工作由 intsetRemove 操作， 它先调用 intsetSearch 找到需要被删除的元素在 contents 数组中的索引， 然后使用内存移位操作，将目标元素从内存中抹去， 最后，通过内存重分配，对 contents 数组的长度进行调整。

#### 小结
- Intset 用于有序、无重复地保存多个整数值，会根据元素的值，自动选择该用什么长度的整数类型来保存元素。
- 当一个位长度更长的整数值添加到 intset 时，需要对 intset 进行升级，新 intset 中每个元素的位长度，会等于新添加值的位长度，但原有元素的值不变。
- 升级会引起整个 intset 进行内存重分配，并移动集合中的所有元素，这个操作的复杂度为 O(N) 。
- Intset 只支持升级，不支持降级。
- Intset 是有序的，程序使用二分查找算法来实现查找操作，复杂度为 O(lgN) 。

