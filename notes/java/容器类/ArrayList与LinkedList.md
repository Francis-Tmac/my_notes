# 关注点
数据结构，插入，删除，扩容，查找，实现的功能

## Collection
### Set
不能包含重复元素，Set判断两个对象不是使用==来判断，是使用equals方法，新加入的元素会与已有的元素判断equals比较返回false则加入，否则拒绝加入；所以使用Set的时候有两点需要注意：1.放入的对象要实现equals方法；2.对set的构造函数中，传入的Collection参数不能包含重复的元素。

### List
List代表元素有序，可重复的集合，集合中每个元素都有对应的顺序索引，允许加入重复元素，通过索引指定元素的位置，实现有ArrayList,Vector,Queue。

### Queue
queue用于模拟队列这种数据结构，先进先出。

### 对比
ArrayList 实现List 接口和RandomAccess 接口，LinkedList 实现List 接口和 Queue 接口；
RandomAccess 实现了这个接口的类支持快速（通常是固定时间）随机访问

### ArrayList:
```$xslt
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{}

transient Object[] elementData;

```
transient： 不被序列化。密码，银行卡等信息防止泄露，只允许存在内存中。

ArrayList 中的关键字段 elementData 使用了 transient 关键字修饰，这个关键字的作用是，让它修饰的字段不被序列化；

ArrayList 内部提供了两个私有方法 writeObject 和 readObject 来完成序列化和反序列化。

#### LinkedList
LinkedList 是一个继承自 AbstractSequentialList 的双向链表，因此它也可以被当作堆栈、队列或双端队列进行操作。

```$xslt
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
ArrayList 在添加元素的时候如果不涉及到扩容，性能在两种情况下（中间位置新增元素、尾部新增元素）比 LinkedList 好很多，只有头部新增元素的时候比 LinkedList 差，因为数组复制的原因。

当然了，如果涉及到数组扩容的话，ArrayList 的性能就没那么可观了，因为扩容的时候也要复制数组。

#### 扩容
ArrayList: 一旦在添加元素的时候，发现容量用满了 s == elementData.length，就按照原来数组的 1.5 倍（oldCapacity >> 1）进行扩容。扩容之后，再将原有的数组复制到新分配的内存地址上 Arrays.copyOf(elementData, newCapacity)。

序列化的时候，如果把整个数组都序列化的话，是不是就多序列化了 4 个内存空间。当存储的元素数量非常非常多的时候，闲置的空间就非常非常大，序列化耗费的时间就会非常非常多。

#### ArrayList 和 LinkedList 新增元素时究竟谁快？

##### ArrayList
ArrayList 新增元素有两种情况，一种是直接将元素添加到数组末尾，一种是将元素插入到指定位置
先判断是否需要扩容，然后直接通过索引将元素添加到末尾

指定位置插入：先检查插入的位置是否在合理的范围之内，然后判断是否需要扩容，再把该位置以后的元素复制到新添加元素的位置之后，最后通过索引将元素添加到指定的位置。这种情况是非常伤的，性能会比较差。

##### LinkedList
```$xslt
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```
添加到最后一个位置

#### 删除
ArrayList

ArrayList 删除元素的时候，有两种方式，一种是直接删除元素（remove(Object)），需要直先遍历数组，找到元素对应的索引；一种是按照索引删除元素（remove(int)）。

从源码可以看得出，只要删除的不是最后一个元素，都需要数组重组。删除的元素位置越靠前，代价就越大。

LinkedList
先检查索引，再调用 node(int) 方法（ 前后半段遍历，和新增元素操作一样）找到节点 Node，然后调用 unlink(Node) 解除节点的前后引用，同时更新前节点的后引用和后节点的前引用：

#### ArrayList 和 LinkedList 遍历元素时究竟谁快

由于 ArrayList 是由数组实现的，所以根据索引找元素非常的快，一步到位。

LinkedList 使用 for 循环遍历时效率极低，因为每一个get 操作都需要又遍历一次半个链表。
而用 foreach 或迭代器遍历是使用 链表元素的 next 指针遍历。

LinkedBlockingQueue 和 LinkedList
LinkedBlockingQueue 单向列表实现BlockingQueue 接口
LinkedList 双向列表实现 List Deque 接口。