## GC 是什么时间，对什么东西，做了什么事情？
### 什么时间
GC主要是回收堆中的内存，当新生代的Eden区满时会触发 Minor GC，当老年代满时会触发 Full GC.
### 什么东西
通过GC Roots 查找的引用链中，没有被引用到的对象。而且经过第一次标记、清理后，仍然没有复活的对象。
### 做什么事情
不同的垃圾收集器和区域做的事情都不同，新生代中主要使用标记-复制算法。老年代中主要使用标记整理，标记-清理算法。
jdk8 默认使用 paralle 收集器，
CMS 收集器使用标记-清除算法，它的收集包括四个阶段 初始标记-》并发标记-》重新标记-》并发清理
初始标记和重新标记需要stw. CMS 的停顿时间很少，但是会产生大量的内存碎片


### 内存泄露
已经使用完的对象没有及时的释放，申请内存后，无法释放已申请的内存空间，内存泄露堆积后会变成内存溢出。

### 内存溢出
新创建的对象无法申请足够的内存空间，出现 out of memory.申请的内存超出了系统能够给到的。