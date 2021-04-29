## 线程池原理
线程是一个重量级的资源，创建，启动一级销毁都是比较耗费系统资源的。线程数量和系统性能是一种抛物线关系。
- 任务队列：用于缓存提交的任务
- 线程数量管理功能：
    - 初始线程数量 init
    - 最大线程数量 core
    - 核心线程数量 max
    - init  ≤ core ≤ max
-  任务拒绝策略：如果线程数量达到 上线且任务队列已满，则需要有拒绝 策略通知任务提交者
- 线程工厂：定制化线程，设置守护线程，线程名称等
- QueueSize: 任务队列主要存放提交的 Runnable
- keepedalive: 决定线程

## ThreadPoolExecutor
###  runWorker
- `ThreadPoolExecutor#runWorker(Worker w)`
    - runWorker 在 Worker 中的 run 方法中被调用，而 runWorker 方法中会循环调用 task 的run 方法
    - 此方法中有一个循环，在其内部会获取任务执行，若获取不到会阻塞或者获取超时会放回。
    - `getTask()---> `
```  java
public class ThreadPoolExecutor extends AbstractExecutorService {
    
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();  
        Runnable task = w.firstTask;  
        w.firstTask = null;  
        w.unlock(); // allow interrupts  
        boolean completedAbruptly = true;  
        try {  
            // getTask 获取任务执行,获取任务为 null 则退出循环
            // getTask 方法中获取任务当为核心线程并且 allowCoreThreadTimeOut=false 时阻塞获取
            // 当线程数量大于核心线程数量时设置 keepAliveTime 时间获取
            while (task != null || (task = getTask()) != null) {  
                w.lock();  
                // If pool is stopping, ensure thread is interrupted;  
                // if not, ensure thread is not interrupted.  This 
                // requires a recheck in second case to deal with 
                // shutdownNow race while clearing interrupt  
                if ((runStateAtLeast(ctl.get(), STOP) ||  
                     (Thread.interrupted() &&  
                      runStateAtLeast(ctl.get(), STOP))) &&  
                    !wt.isInterrupted())  
                    wt.interrupt();  
                try {  
                    beforeExecute(wt, task);  
                    Throwable thrown = null;  
                    try {  
                        task.run();  
                    } catch (RuntimeException x) {  
                        thrown = x; throw x;  
                    } catch (Error x) {  
                        thrown = x; throw x;  
                    } catch (Throwable x) {  
                        thrown = x; throw new Error(x);  
                    } finally {  
                        afterExecute(task, thrown);  
                    }  
                } finally {  
                  task = null;  
                  w.completedTasks++;  
                  w.unlock();  
                } 
            } 
            completedAbruptly = false;  
        } finally {  
            // 
            processWorkerExit(w, completedAbruptly);  
        }
    }
    
    private Runnable getTask() {  
        boolean timedOut = false; // Did the last poll() time out?  
  
        for (;;) {  
            int c = ctl.get();  
            int rs = runStateOf(c);  
  
            // Check if queue empty only if necessary.  
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {  
                decrementWorkerCount();  
                return null;  
            }  
  
            int wc = workerCountOf(c);  
  
            // Are workers subject to culling?  
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;  
  
           if ((wc > maximumPoolSize || (timed && timedOut))  
                && (wc > 1 || workQueue.isEmpty())) {  
                if (compareAndDecrementWorkerCount(c))  
                    return null;  
                continue;  
           }  
  
           try {  
                // poll() 非阻塞方法，超过keepAliveTime 则返回空
                // 因为线程处理完任务会持续调用 getTask方法，当 timed= wc > corePoolSize 时，会使用keepAliveTime 进行有效时长获取任务。
                // 当timed =  false  时调用阻塞方法，持续获取任务
                // tips: 当线程数量达到 coreSize 后，并且allowCoreThreadTimeOut = false ,是否后续线程池的大小不会低于 coreSize
                Runnable r = timed ?  
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :  
                    workQueue.take();  
                if (r != null)  
                    return r;  
                timedOut = true;  
           } catch (InterruptedException retry) {  
            timedOut = false;  
           }
        }
    }
    
    
}
```

### runState
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static int ctlOf(int rs, int wc) { return rs | wc; }
}
```
####  线程池运行状态 runState
- RUNNING: 接受新任务，并处理队列任务
- SHUTDOWN: 不接受新任务，但处理排队的任务
- STOP: 不接受新任务，不处理排队任务，和中断进行中的任务
- TIDYING: 所有任务都已终止，workerCount为零，线程转换为状态TIDYING 将运行Terminated（）挂钩方法
- TERMINATED: Terminate（）已完成

AtomicInteger 类型的ctl代表了ThreadPoolExecutor中的控制状态，它是一个复核类型的成员变量，是一个原子整数，借助高低位包装了两个概念：
1. workerCount：线程池中当前活动的线程数量，占据ctl的低29位；
2. runState：线程池运行状态，占据ctl的高3位，有RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED五种状态
3. running -1<<29  -1的32位为32位1，向左移29位后为3个1  `11100000 +24_0`

### execute
- `ThreadPoolExecutor#execute(Runnable command)`
- `ThreadPoolExecutor#addWorker(Runnable firstTask, boolean core)`
    - 先进行容量判断后在新建 线程
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    public void execute(Runnable command) {  
     if (command == null)  
        throw new NullPointerException();  
     int c = ctl.get();  
     // 当新增任务时正在执行线程，小于核心线程。
     // 会直接新增线程处理 firstTask 并且直接返回
     if (workerCountOf(c) < corePoolSize) {  
        if (addWorker(command, true))  
            return;  
        c = ctl.get();  
      }  
     // 这里正在执行线程大于等于核心线程数的情况，会做一个 double-check
     // 如果需要会新建线程 但不会有 firstTask 
     if (isRunning(c) && workQueue.offer(command)) {  
         int recheck = ctl.get();  
         // 如果当前线程状态不是 running ，并且能从队列移除 command 成功
         // 拒绝任务
         if (! isRunning(recheck) && remove(command))  
            reject(command);  
         // 当工作线程worker 数目为0 时，尝试添加新的 worker 线程，但是不携带任务
         // 当 ctl 为0 是意味着 线程池状态为 shutDown，并且工作线程数量为 0
         else if (workerCountOf(recheck) == 0)  
             
            addWorker(null, false);  
     }  
     else if (!addWorker(command, false))  
        reject(command);  
    }
    
   private boolean addWorker(Runnable firstTask, boolean core) {  
    retry:  
    for (;;) {  
         int c = ctl.get();  
         // 获取状态
         int rs = runStateOf(c);  
  
          // Check if queue empty only if necessary.  
          if (rs >= SHUTDOWN &&  
            ! (rs == SHUTDOWN &&  
               firstTask == null &&  
               ! workQueue.isEmpty()))  
            return false;  
         // 用循环CAS操作来将线程数加1
         for (;;) {  
             int wc = workerCountOf(c);  
             // core = true 是添加核心线程，false 为添加最大线程
             // 线程超标则返回
             if (wc >= CAPACITY ||  
                    wc >= (core ? corePoolSize : maximumPoolSize))  
                    return false;  
             // 添加成功退出循环
             if (compareAndIncrementWorkerCount(c))  
                    break retry;  
              c = ctl.get(); // Re-read ctl  
              // 如果线程池的状态发生变化 返回到 retry 外层循环
              if (runStateOf(c) != rs)  
                    continue retry;  
              // else CAS failed due to workerCount change; retry inner loop  
          }  
        }  
     
     // 新建线程，并加入到线程池 works 中
     boolean workerStarted = false;  
     boolean workerAdded = false;  
     Worker w = null;  
     try {  
         w = new Worker(firstTask);  
         final Thread t = w.thread;  
         if (t != null) {  
            final ReentrantLock mainLock = this.mainLock;  
            mainLock.lock();  
            try {  
                // Recheck while holding lock.  
                // Back out on ThreadFactory failure or if 
                // shut down before lock acquired.  int rs = runStateOf(ctl.get());  
  
                 if (rs < SHUTDOWN ||  
                    (rs == SHUTDOWN && firstTask == null)) {  
                    if (t.isAlive()) // precheck that t is startable  
                      throw new IllegalThreadStateException();  
                    workers.add(w);  
                    int s = workers.size();  
                    if (s > largestPoolSize)  
                        largestPoolSize = s;  
                    workerAdded = true;  
                    }  
            } finally {  
                mainLock.unlock();  
            }  
            if (workerAdded) {  
                t.start();  
                workerStarted = true;  
            }  
        }  
    } finally {  
        if (! workerStarted)  
            addWorkerFailed(w);  
  }  
    return workerStarted;  
}    
    
}

```

###  Worker
worker 类继承了 AQS 

### question
- 是否所有的任务都会放到任务队列中，线程池中的队列总任务队列中消费队列。
    - 不是当正在执行线程小于核心线程数时，新提交的任务会直接有新创建的线程执行
- 守护线程和用户线程的区别
    - 守护线程通过 `Thread.setDaemon(true)` 设置，必须在 `Thread.start()` 之前调用
    - 唯一的区别是判断虚拟机(JVM)何时离开，Daemon 是为其他线程提供服务，如果 全部的 User Thread 已经撤离，Daemon 没有可服务的线程，JVM 撤离。
    - JVM 的垃圾回收线程是一个守护线程，当所有线程已经撤离，不在产生垃圾，JVM 自然会关闭
- 什么是阻塞队列？ 实现原理是什么？
    - 阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。
    - 在队列为空时，获取元素的线程会等待队列变为非空。
    - 当队列满时，存储元素的线程会等待队列可用。
    - put方法 take方法 阻塞方法， offer 和 poll 是带时间的阻塞方法。
    - ArrayBlockingQueue 内部维护两个 `Condition notEmpty,notFull` 当 调用 `take` 方法队列为空时调用 `notEmpty.await()` 阻塞线程，在有线程向队列中添加任务后会调用 `notEmpty.signal` 唤醒线程去获取任务。


## reactor
与线程池不同的是每一个队列都有一个任务队列，并且不是阻塞的。
reactor 中的线程有两种定义一种处理IO事件。