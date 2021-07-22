## 悲观锁
对数据的修改持有悲观态度的并发控制方式，任务每次对数据的修改都会同时有其他线程一起对该数据进行修改。
所以需要在对该数据访问的时候加上锁，其他的线程修改该数据时会被阻塞。
适合写入操作比较频繁的场景，如果出现大量的读取操作，每次读取的时候都会加锁，这样会增加锁的开销，降低系统的吞吐量
## 乐观锁
相对于悲观锁而言，认为不会存在并发冲突，只在每次修改完数据后对数据进行冲突检测。

比较适合读取操作比较频繁的场景，如果出现大量的写入操作数据发生冲突的可能性就会增大，为了 保证数据一致性 ，需要不断重新获取数据 ，增大查询操作，降低吞吐量

乐观锁适用于写操作比较少的情况，冲突真的很少发生的时候 ，可以省去锁的开销，加大了系统的整体吞吐量。如果经常发生冲突，需要悲观锁不断的retry，也就是冲突比较严重的 情况下，所以采用悲观锁

在循环打印中，如果用乐观锁实现会导致CPU 使用飙升，乐观锁一般都是在死循环中 尝试去修改值。



## StampedLock



## ReentrantLock-重入锁
- 该锁可以支持一个线程对资源重复加锁。
- 公平锁先对锁进行请求一定先获取到锁，非公平锁没有时间上的先后顺序

## AQS-AbstractQueuedSynchronizer
队列同步器，用来构建锁或者其他同步组件的基础框架。AQS 内部通过内部类 NODE 构成FIFO 的同步队列来完成线程获取释放锁的排队工作，同事利用内部类 ConditionObject  构建等待队列。
```java
static final class Node {  
      /** Marker to indicate a node is waiting in shared mode */  
      static final Node SHARED = new Node();  
      /** Marker to indicate a node is waiting in exclusive mode */  
      static final Node EXCLUSIVE = null;  

      /** 
      *在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点，
      *其结点的waitStatus为CANCELLED，即结束状态，进入该状态后的结点将不会再变化。 
      */  
      static final int CANCELLED =  1;  
      /** 处于唤醒状态，只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行*/  
      static final int SIGNAL = -1;  
      /** 
      *与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，
      *当其他线程调用了Condition的signal()方法后，
      *CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁 
      */  
      static final int CONDITION = -2;  
      /**  
      * 值为-3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。 
      */
      static final int PROPAGATE = -3;
    
    
}

```
- AQS 作为基础组件，对于锁的实现存在两种不同的模式，即共享模式（如Semaphore） 和 独占模式（如ReentrantLock），无论是共享模式还是独占模式的实现类，都是维护一个虚拟的同步队列，当请求的线程超过现有模式的限制时，会将线程包装成Node 节点。
- AQS 采用模板方法模式，内部出了提供并发操作核心方法以及同步队列操作外，还提供了一些模板方法让子类自己实现。比如共享模式与独占模式，这两种加锁与解锁方式是不一样的，但是AQS只关注内部公共方法实现并不关心外部不同模式的实现

- 同步队列
- 同步状态


#### 非公平锁
1. lock -> 尝试修改state 状态
2. acquire -> tryAcquire 尝试修改state 状态，并且做可重入锁判断
3. 添加到队列后，如果前节点是头节点也会尝试修改 state 状态。
 
获取锁的方法 `ReentrantLock#lock()` `ReentrantLock#tryLock()`
##### ReentrantLock#lock()
ReentrantLock#lock() ---> Sync#lock() ---> NonfairSync#lock() 

##### ReentrantLock#tryLock()
只有一次尝试获取锁失败则返回，没有进入同步队列
ReentrantLock#lock() ---> Sync#nonfairTryAcquire()
```java
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            // 第一次尝试获取锁
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                // 获取锁失败后调用 acquire 方法
                acquire(1);
        }
        // AbstractQueuedSynchronizer#acquire() 为抽象方法将调用子类实现的 tryAcquire()方法
        protected final boolean tryAcquire(int acquires) {
            // 调用的 nonfairTryAcquire 方法为
            return nonfairTryAcquire(acquires);
        }
    }
```
- Sync
```java
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 尝试获取锁
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
    }
```

- acquire()
```java
    public abstract class AbstractQueuedSynchronizer
        extends AbstractOwnableSynchronizer
        implements java.io.Serializable {

            public final void acquire(int arg) {
                // tryAcquire 会调用到 Sync#nonfairTryAcquire() 尝试获取锁
                // addWaiter(Node.EXCLUSIVE) 将线程加入到同步队列中
                // 加入后会判断 前驱节点是否为头结点，若是则再次尝试获取锁

                if (!tryAcquire(arg) &&
                    acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                    selfInterrupt();
            }
            
            final boolean acquireQueued(final Node node, int arg) {
                boolean failed = true;
                try {
                    boolean interrupted = false;
                    // 进入自旋
                    for (;;) {
                        final Node p = node.predecessor();
                        // 再次判断后尝试获取锁
                        if (p == head && tryAcquire(arg)) {
                            setHead(node);
                            p.next = null; // help GC
                            failed = false;
                            return interrupted;
                        }
                        // 如果前驱结点不是head，判断是否挂起线程
                        // 
                        // parkAndCheckInterrupt() 将当前线程挂起
                        if (shouldParkAfterFailedAcquire(p, node) &&
                            parkAndCheckInterrupt())
                            interrupted = true;
                    }
                } finally {
                    if (failed)
                        cancelAcquire(node);
                }
            }


        }
```

shouldParkAfterFailedAcquire() 方法的作用是判断当前结点的前驱结点是否为SIGNAL状态(即等待唤醒状态)，如果是则返回true。
如果结点的ws为CANCELLED状态(值为1>0),即结束状态，则说明该前驱结点已没有用应该从同步队列移除，执行while循环，
直到寻找到非CANCELLED状态的结点。倘若前驱结点的ws值不为CANCELLED，也不为SIGNAL(当从Condition的条件等待队列转移到同步队列时，
结点状态为CONDITION因此需要转换为SIGNAL)，那么将其转换为SIGNAL状态，等待被唤醒。

#### 公平锁
1. 如果队列为空，会尝试修改state 状态。再做重入锁判断

ReentrantLock 实现可重入锁，对state 的期望值是0时会获取锁，每次对state加一
Semaphore 作为互斥锁是没有实现可重入锁，对 state 的期望大于0时获取锁，每次对state 减1。

### ReentrantLock 和 Sychronized 使用场景
- ReentrantLock 中有公平锁的实现，Sychronized 中进入waitSet后每次获取锁都是随机的
- ReentrantLock 中可以有多个等待队列，灵活调用。Sychronized只有一个 waitSet 而且每次获取锁的都是随机。
- ReentrantLock 有超时机制等待可中断，获取不到锁会释放线程资源，当大量的线程使用 Sychronized 获取不到线程阻塞后，对线程资源的压力很大。

- ReentrantLock 更大可能造成死锁，当获取锁后没释放锁造成了死锁， Sychronized 在执行完代码块后就隐式的释放对象锁。
- Sychronized 与生俱来的锁对象头的，不会产生多余的线程。ReentrantLock 显示创建锁，当系统中有成百上千个 ReentrantLock 时对内存的资源消耗是不可忽视的。