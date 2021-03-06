## jvm 参数设置
-X：非标准选项
以 -X 开头的这些选项是非标准选项，是特定于 Java HotSpot 虚拟机的通用选项，不保证所有 JVM 的实现都支持它们，而且还会发生变化。

-XX：高级选项
以 -XX 开头的选项是高级选项，高级选项不建议随意使用。这些是开发人员用于调优 Java HotSpot 虚拟机操作的特定区域的选项，这些选项通常具有特定的系统需求，并且可能需要特权访问系统配置参数。它们也不保证得到所有 JVM 的实现的支持，并且可能会发生变化。


-verbose
打印加载类的详细信息

-verbose:gc
打印虚拟机中GC的详细情况:显示最忙和最空闲收集行为发生的时间、收集前后的内存大小、收集需要的时间等

-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs/java.hprof
虚拟机在出现内存溢出异常时Dump 出当前的内存堆转储快照

-XX:+TraceClassLoading
打印加载类的详细信息

-XX:+PrintGCDetails
详细了解GC中的变化

-XX:+PrintGCDateStamps
了解垃圾收集发生的时间,自JVM启动以后以秒计量

-Xloggc:/home/admin/logs/gc.log
GC日志文件的路径
涉及堆相关的参数

-server
server模式下,新生代选择的是并行GC,旧生代选择的是并行GC
client模式下,新生代选择的是串行GC,旧生代选择的是串行GC

-Xms2g  
堆的初始值2g memory start

-Xmx2g  memory  max
堆的最大值2g
PS:避免在运行时频繁调整Heap的大小,通常-Xms与-Xmx的值设成一样

-XX:MinHeapFreeRation=
空余堆内存小于MinHeapFreeRation时,JVM会增大Heap到-Xmx指定的大小

-XX:MaxHeapFreeRation=
空余堆内存大于MaxHeapFreeRation时,JVM会减小heap的大小到-Xms指定的大小

-Xmn1g memory new 
新生代堆大小1G

-XX:SurvivorRatio=8
默认32:1:1
新生代的Eden区:From区:To区的比例为8:1:1

-XX:PermSize= (JDK7)
永久代大小

-XX:MaxPermSize= (JDK7)
永久代MAX大小

-XX:MetaspaceSize= (JDK8)
代替PermSize,元空间大小

-XX:MaxMetaspaceSize= (JDK8)
代替MaxPermSize,元空间最大值

回收算法
标记-清理

直接清理
效率高
产生内存碎片
标记-复制

两块内存区域,S1,S2
不存在内存碎片
占用双倍空间
标记-整理

在标记-清理之后,让剩余存活的对象都向一端移动,并更新引用其对象的指针
效率低
不产生碎片
分代收集算法

把Java堆分新生代和老年代
在新生代用复制算法
在老年代用标记-清理或标记-整理算法
垃圾清理类型

-XX:+UseSerialGC
Serial GC
串行
使用简单的标记、清除、压缩方法对年轻代和年老代进行垃圾回收,即Minor GC和Major GC
适用于CPU配置较低,内存占用较少的单独Client应用模式

-XX:+UseParallelGC
Parallel GC
并行
收集方式同Serial GC一样
产生N个线程来进行年轻代的垃圾收集
N默认=系统CPU核数

-XX:ParallelGCThreads=
设置Parallel GC线程数量

-XX:+UseParallelOldGC
Parallel Old GC
方式同Parallel GC
主要是年轻代和年老代垃圾回收时都使用多线程收集

-XX:+UseConcMarkSweepGC
并发标记清除（CMS）收集器
短暂停顿并发收集器

CMS-initial-mark：初始标记阶段(stop the world)
CMS-concurrent-mark : 和应用线程并发执行,标记可达的对象
CMS-concurrent-preclean(CMS-concurrent-preclean-start,CMS-concurrent-preclean) : 预清理(标记和应用线程是并发执行的,有些对象的状态在标记后会改变)
CMS-concurrent-abortable-preclean : 进一步的预清理,减少Rescan阶段时间.使用到参数(-XX:CMSMaxAbortablePrecleanTime)
Rescan阶段(stop the world) : 暂停应用线程,对对象进行重新扫描并标记
CMS-concurrent-sweep : 并发的垃圾清理
CMS-concurrent-reset : 下一次CMS GC重置相关数据结构
对年老代进行垃圾收集
年轻代使用的算法和Parallel一样
适用于不能忍受长时间停顿要求快速响应的应用
!!!FULL GC
concurrent-mode-failure : CMS GC时,有新的对象要进入年老代,但是年老代空间不足
promotion-failed : Young GC时,存活对象从Eden区到Survivor区,但是Survivor区空间不足,需要到年老代,年老代空间也不足

-XX:CMSMaxAbortablePrecleanTime=
CMS GC在concurrent-abortable-preclean阶段使用的参数
当abortable-preclean阶段执行达到这个时间时才会结束

-XX:+CMSClassUnloadingEnabled
避免Perm区满引起的Full GC,开启CMS回收Perm区

-XX:CMSInitiatingOccupancyFraction=
此参数值是一个比例(参考下一条)

-XX:+UseCMSInitiatingOccupancyOnly
如果设置了此参数
只有当年老代占用达到了-XX:CMSInitiatingOccupancyFraction参数所设定的比例时才会触发CMS

-XX:ParallelCMSThreads=
设置CMS收集器的线程数量

-XX:+UseG1GC
G1垃圾收集器
属于分代收集器
JDK7+,长远目标是代替CMS收集器
并行的、并发的和增量式压缩短暂停顿的垃圾收集器
缩短处理超大堆时产生的停顿
相对于CMS的优势而言是内存碎片的产生率大大降低
G1收集器和其他的收集器运行方式不一样,不区分年轻代和年老代.它把堆空间划分为多个大小相等的区域,当进行垃圾收集时,它会优先收集存活对象较少的区域
采用Remembered Set来避免全堆扫描

-XX:MaxGCPauseMillis=200
设置GC的最大暂停时间为200ms
如果MaxGCPauseMillis设置的过小,那么GC就会频繁,吞吐量就会下降
如果MaxGCPauseMillis设置的过大,应用程序暂停时间就会变长

-XX:G1HeapRegionSize=
设置的G1区域的大小
值是2的幂,范围是1MB到32MB之间
-Dsun.rmi.dgc.server.gcInterval=2592000000
-Dsun.rmi.dgc.client.gcInterval=2592000000

存在RMI调用时,默认会每分钟执行一次System.gc,可以通过此参数来设置大点的间隔
堆外内存相关参数

-XX:MaxDirectMemorySize=
指定了DirectByteBuffer分配空间限额
DirectByteBuffer通过内存映射,使Java应用进程直接访问与文件相关联的虚拟地址空间,减少了文件拷贝带来的开销,提高了文件读取效率
默认会调用Full GC来回收

-XX:+DisableExplicitGC
禁用Full GC显示调用(禁用System.GC)
会出现OOM: 当系统各方面性能良好,无Full GC且DirectByteBuffer所占用的空间大于-Xmx分配的空间.
因为DirectByteBuffer会不断地在Native堆分配空间,它的引用进入了Old区,Old区保存大量的引用,而不能被回收,最终会导致Native堆空间不足
可以通过设置-XX:MaxDirectMemorySize=几十M,使OOM来得更早一些

-XX:+ExplicitGCInvokesConcurrent
使用此参数,在进行堆外内存Full GC时,使用CMS并发GC,减少单纯的Full GC的卡顿时间,提高GC效率
other options
-Dsun.net.client.defaultConnectTimeout=10000
-Dsun.net.client.defaultReadTimeout=30000
设置默认的Http请求的连接超时时间和读取超时时间

-Dfile.encoding=UTF-8
设置文件流编码

-agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n
开放8000端口,支持远程DEBUG

### 某台机器上的服务挂掉
```
-Xms1024m -Xmx1024m -Xmn1024m
```
服务报错信息
```
java.lang.OutOfMemoryError: Java heap space

### Error querying database. Cause: java.sql.SQLException: Java heap space
```

#### 现象：
该机器上的日志停留在半个小时以前，服务进程已不存在


疑问：当时进程是否存在，如果存在jvm 内的堆状态是怎么样


上游核心搞传的 repaytype=3 理赔的消息， 生成的汇总标识为空，用空的去查所有的，
索引字段命中了几乎全表的数据，大概三百一十万笔记录。

#### 自测

提供一个接口，每调一次就会产生一个 100MB 的大对象


```
    @ResponseBody
    @RequestMapping("/testOom")
    public String testOom(Integer size) throws Exception{
        byte[] allocation1,allocation2,allocation3;
        allocation1 = new byte[size * _100Mb];
        Thread.sleep(100000);
        log.info("生成100MB", allocation1.length);
        return "ok";
    }

```
##### 设置1 

```
-Xms512m -Xmx512m -Xmn512m
```
老年代几乎没有内存，每次对象都是新生代中创建，当新生代内存满后，新的调用将会报错，日志会打印oom


```
|2020-09-10 13:07:19.178|ERROR||http-nio-2898-exec-5|c.x.fundhub.web.handler.GlobalExceptionHandler|throwableHandler[28]||throwable handler, throwable:{}
org.springframework.web.util.NestedServletException: Handler dispatch failed; nested exception is java.lang.OutOfMemoryError: Java heap space
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1053)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1005)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:908)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:665)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:882)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:750)
```
jvm 中常常发生 Full GC 

```
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
25600.0 30720.0  0.0    0.0   462336.0 461407.2   512.0      469.6    79528.0 76172.9 9904.0 9312.5      8    0.587  22      6.222    6.809
25600.0 30720.0  0.0    0.0   462336.0 461407.2   512.0      469.6    79528.0 76172.9 9904.0 9312.5      8    0.587  22      6.222    6.809
25600.0 30720.0  0.0    0.0   462336.0 461544.3   512.0      469.6    79528.0 76172.9 9904.0 9312.5      8    0.587  22      6.222    6.809
25600.0 30720.0  0.0    0.0   462336.0 462006.7   512.0      469.6    79528.0 76172.9 9904.0 9312.5      8    0.587  22      6.222    6.809
25600.0 30720.0  0.0    0.0   462336.0 139325.8   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139667.4   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 139675.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 242411.9   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 242469.7   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 242547.2   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 242820.2   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 346004.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 346004.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 346004.0   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 346025.8   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 448534.3   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 449016.9   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 449016.9   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  23      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 449683.7   512.0      468.7    79528.0 76195.0 9904.0 9315.2      8    0.587  24      6.387    6.975
25600.0 30720.0  0.0    0.0   462336.0 447545.6   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 447556.2   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 447556.2   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 447556.2   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 447787.4   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 448475.3   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 448533.0   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 448651.1   512.0      468.7    79528.0 76194.4 9904.0 9313.6      8    0.587  25      6.904    7.491
25600.0 30720.0  0.0    0.0   462336.0 446907.4   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 446911.1   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 447159.1   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 447162.3   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 447162.3   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 447162.3   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 447162.3   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  27      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 449026.7   512.0      468.7    79528.0 76199.6 9904.0 9313.6      8    0.587  28      7.217    7.804
25600.0 30720.0  0.0    0.0   462336.0 447074.1   512.0      468.7    79528.0 76202.8 9904.0 9313.6      8    0.587  29      7.562    8.150
25600.0 30720.0  0.0    0.0   462336.0 447076.5   512.0      468.7    79528.0 76202.8 9904.0 9313.6      8    0.587  29      7.562    8.150
25600.0 30720.0  0.0    0.0   462336.0 447076.5   512.0      468.7    79528.0 76202.8 9904.0 9313.6      8    0.587  29      7.562    8.150
```

##### 设置-2

```
-Xms512m -Xmx512m -Xmn126m
```
老年代内存为 390M 每次对象创建在老年代中， 发生full GC 的次数相对减少很多，经常在新生代中发生GC 
```
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
7680.0 7680.0  0.0   1904.1 113152.0  9015.1   395264.0   54781.8   77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0  9015.1   395264.0   54781.8   77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0  9015.1   395264.0   54781.8   77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0  9015.1   395264.0   54781.8   77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 44705.9   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 44705.9   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 44705.9   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 44705.9   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 44705.9   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 44705.9   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 45271.8   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 45271.8   395264.0   157181.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 47036.5   395264.0   259581.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 47036.5   395264.0   259581.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 47036.5   395264.0   259581.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 47036.5   395264.0   259581.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 47036.5   395264.0   259581.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 47359.9   395264.0   259581.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 48491.5   395264.0   361981.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 48491.5   395264.0   361981.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 48491.5   395264.0   361981.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7680.0  0.0   1904.1 113152.0 48491.6   395264.0   361981.9  77016.0 74076.5 9728.0 9189.7     37    0.660   3      0.543    1.203
7680.0 7168.0 848.2   0.0   114176.0 102400.0  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 104158.1  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 104158.1  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 104158.1  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 106632.4  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 110516.4  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 110516.4  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 110516.4  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2   0.0   114176.0 110516.4  395264.0   363634.1  77912.0 74969.6 9856.0 9309.5     38    0.679   3      0.543    1.222
7680.0 7168.0 848.2  992.2  114176.0 112251.9  395264.0   363634.1  77912.0 75031.8 9856.0 9317.6     39    0.697   4      0.543    1.240
7680.0 7168.0  0.0    0.0   114176.0 113246.9  395264.0   344212.7  77912.0 74665.0 9856.0 9258.3     40    0.724   5      1.218    1.942
7680.0 7168.0  0.0    0.0   114176.0 113403.8  395264.0   344212.7  77912.0 74665.0 9856.0 9258.3     40    0.724   5      1.218    1.942
7680.0 7168.0  0.0    0.0   114176.0 113403.8  395264.0   344212.7  77912.0 74665.0 9856.0 9258.3     40    0.724   5      1.218    1.942
7680.0 7168.0  0.0    0.0   114176.0 113403.8  395264.0   344212.7  77912.0 74665.0 9856.0 9258.3     40    0.724   5      1.218    1.942
7680.0 7168.0  0.0   352.0  114176.0 114176.0  395264.0   344392.9  78168.0 75003.7 9856.0 9301.5     41    0.737   6      1.218    1.955
7680.0 7168.0  0.0    0.0   114176.0 104428.8  395264.0   344202.8  78168.0 75003.7 9856.0 9301.5     42    0.747   7      1.674    2.420
7680.0 7168.0  0.0    0.0   114176.0 104772.0  395264.0   344202.8  78168.0 75003.7 9856.0 9301.5     42    0.747   7      1.674    2.420
7680.0 7168.0  0.0    0.0   114176.0 104772.0  395264.0   344202.8  78168.0 75003.7 9856.0 9301.5     42    0.747   7      1.674    2.420
7680.0 7168.0  0.0    0.0   114176.0 103876.5  395264.0   344309.8  78168.0 75008.5 9856.0 9301.5     44    0.776   9      2.092    2.868
```
但是服务进程没有挂掉

##### 设置-3


```
@ResponseBody
    @RequestMapping("/testOom")
    public String testOom() throws Exception{
        byte[] allocation1,allocation2,allocation3;

        Thread.sleep(100000);
        for(int i = 0 ; i < 15 ; i++){
            byte[] allocation = new byte[_100Mb];
            log.info("生成100MB i: {}, allocation.length: {}", i, allocation.length);
        }
        Thread.sleep(120000);

        return "ok";
    }
```

```
-Xms512m -Xmx512m -Xmn126m
```


```
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC    CCSU       YGC   YGCT    FGC    FGCT     GCT
3584.0 4096.0  0.0    32.0  120832.0 109170.4  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 109943.8  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 109943.8  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 109943.8  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 109943.8  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 110572.2  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 113109.7  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 113109.7  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 113714.0  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 113714.0  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 113714.0  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.6  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.6  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.6  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.6  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.6  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.7  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.7  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.8  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
3584.0 4096.0  0.0    32.0  120832.0 114519.8  395264.0   364280.3  77912.0 75011.7 9856.0 9299.8     49    0.734   3      0.567    1.301
1536.0 1536.0  32.0   0.0   125952.0 105567.2  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 105567.2  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 105682.5  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 108745.4  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 108745.4  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 109171.2  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 112367.0  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 112367.0  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 116132.7  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 118651.8  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 118651.8  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 119071.6  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 119071.6  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 119071.6  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 121603.4  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 122658.8  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 122658.8  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 123288.6  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
1536.0 1536.0  32.0   0.0   125952.0 123288.6  395264.0   364324.3  78168.0 75042.7 9856.0 9299.8     64    0.869   3      0.567    1.436
```
