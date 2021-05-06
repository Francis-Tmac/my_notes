## 无锁概念
加锁是一种悲观策略，无锁是一种乐观策略。
- 加锁：对于并发编程来说，总是认为每次访问共享资源时总是会发生冲突，因此必须对每次操作加锁。
- 无锁：假设每一次对共享资源的访问没有冲突，线程可以不停的执行，无需加锁，无需等待，一旦发现冲突，则采用CAS 技术保证线程的安全性。

### 无锁执行者 CAS
CAS 全称 `Compare And Swap`
`执行函数： CAS(V,E,N)`
- V 表示要更新的变量
- E 表示预期值
- N 表示新值

#### CPU 对CAS 的支持
CAS 是一种系统原语，原语属于操作，由若干条指令组成，用于完成某个功能的过程，也就是说 CAS 是一条CPU 原子指令，不会造成所谓的数据不一致问题。

#### Unsafe 类
Unsafe 类中，其内部方法操作可以像指针一样直接操作内存，Java 中的CAS 依赖于 Unsafe 类的方法，Unfafe 类的所有方法都是 native 修饰的，

``` 
//第一个参数o为给定对象，offset为对象内存的偏移量，通过这个偏移量迅速定位字段并设置或获取该字段的值，
//expected表示期望值，x表示要设置的值，下面3个方法都通过CAS原子指令执行操作。
public final native boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);                                                                                 
public final native boolean compareAndSwapInt(Object o, long offset,int expected,int x);

public final native boolean compareAndSwapLong(Object o, long offset,long expected,long x);


```

- 挂起与恢复
    - 将一个线程进行挂起是通过 park 方法实现的，调用park 后，线程将一直阻塞知道超时或者中断等条件出现
    - Java 对线程的挂起操作被封装在 LockSupport 类中，LockSupport 类中有各种版本 pack 方法，其底层最终都是使用 Unsafe.park() 方法和 Unsafe.unpark() 方法
```java

public class LockSupport {
    public static void park() {  
        UNSAFE.park(false, 0L);  
    }
    
    public static void unpark(Thread thread) {  
        if (thread != null)  
            UNSAFE.unpark(thread);  
    }
}
```
- 内存屏障

```
//在该方法之前的所有读操作，一定在load屏障之前执行完成
public native void loadFence();
//在该方法之前的所有写操作，一定在store屏障之前执行完成
public native void storeFence();
//在该方法之前的所有读写操作，一定在full屏障之前执行完成，这个内存屏障相当于上面两个的合体功能
public native void fullFence();
```

### 原子操作类-Atomic系列


```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // 获取指针类Unsafe
    private static final Unsafe unsafe = Unsafe.getUnsafe();

    //下述变量value在AtomicInteger实例对象内的内存偏移量
    private static final long valueOffset;

    static {
        try {
           //通过unsafe类的objectFieldOffset()方法，获取value变量在对象内存中的偏移
           //通过该偏移量valueOffset，unsafe类的内部方法可以获取到变量value对其进行取值或赋值操作
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
   //当前AtomicInteger封装的int变量value
    private volatile int value;

    public AtomicInteger(int initialValue) {
        value = initialValue;
    }
    public AtomicInteger() {
    }
   //获取当前最新值，
    public final int get() {
        return value;
    }
    //设置当前值，具备volatile效果，方法用final修饰是为了更进一步的保证线程安全。
    public final void set(int newValue) {
        value = newValue;
    }
    //最终会设置成newValue，使用该方法后可能导致其他线程在之后的一小段时间内可以获取到旧值，有点类似于延迟加载
    public final void lazySet(int newValue) {
        unsafe.putOrderedInt(this, valueOffset, newValue);
    }
   //设置新值并获取旧值，底层调用的是CAS操作即unsafe.compareAndSwapInt()方法
    public final int getAndSet(int newValue) {
        return unsafe.getAndSetInt(this, valueOffset, newValue);
    }
   //如果当前值为expect，则设置为update(当前值指的是value变量)
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
    //当前值加1返回旧值，底层CAS操作
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
    //当前值减1，返回旧值，底层CAS操作
    public final int getAndDecrement() {
        return unsafe.getAndAddInt(this, valueOffset, -1);
    }
   //当前值增加delta，返回旧值，底层CAS操作
    public final int getAndAdd(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta);
    }
    //当前值加1，返回新值，底层CAS操作
    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
    //JDK 1.7的源码，由for的死循环实现，并且直接在AtomicInteger实现该方法，
    //JDK1.8后，该方法实现已移动到Unsafe类中，直接调用getAndAddInt方法即可
    public final int incrementAndGet() {
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return next;
        }
    }
 
    //当前值减1，返回新值，底层CAS操作
    public final int decrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, -1) - 1;
    }
   //当前值增加delta，返回新值，底层CAS操作
    public final int addAndGet(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
    }
   
}


```














