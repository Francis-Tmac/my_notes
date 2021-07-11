HashMap 是无序的，迭代HashMap 所得到的的元素顺序并不是他们最初放置的HashMap 的顺序。
迭代顺序可以使插入顺序，也可以是访问顺序。根据链表中元素的顺序可以将 LinkedHashMap 分为：保持插入顺序的和保持访问顺序的 LinkedHashMap，其中
插入顺序是默认的排序

- 本质上，HashMap 和双向链表合二为一即是LinkedHashMap 。额外定义了一个以head 为头节点的双向链表，因此每次put 进来 Entry ，除了将其保存在哈希表中，还会
将其插入到双向链表的尾部。

- 成员变量定义增加两个属性保证迭代顺序，分别是双向链表头结点 header 和标志位 accessOrder 时，表示按照访问顺序迭代；为false 时，表示按照插入顺序迭代。

 