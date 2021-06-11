### question
- 问题的背景
- 解决问题前的现象
- 怎么样解决的
- 解决过程中遇到的难点
- 需要注意的点
- 解决后的现象

## 核心接入系统：
### 接入层
落地授信放款还款数据
上游发给我们的用户授信申请，放款申请还款申请并且已经匹配好这笔申请属于哪个资方。

### 运营配置
#### 标准化的审批流程
- 渠道：一个资方，包含渠道id。

- 行为：是用户一次行为也是公司系统执行的最小单位，对应的是接入层落地到表的一条数据处理。一个行为是包含多个步骤，这个步骤为接入系统执行的最小单元，这些步骤组成了一个单边有向图，步骤是在图中的一个节点，在这个有向图中有初始态中间态终态。

- 步骤：系统执行的最小单元，与资方或者上游的一次交互，主动调用，回调类型。步骤包含：与资方的加解密处理器，发送和接收的报文模板以及解析资方报文后的应该去到的下一个步骤（就是节点在有向图中指向的下一个节点）都存储在step中。

- 步骤细节：调用的时候会解析这个模板，通过执行报文配置的处理器得到最终的发送报文，再通过加解密处理器发送给资方，得到资方的返回报文对报文解析后，判断下一步到哪里去。

- 处理器：从效果上来说就是一次方法的执行，主要存在与发送报文模板中，大多数字段的获取都是通过处理器完成，处理器之间可以相互调用。把代码中的方法，包括类的路径，方法名称，入参，返回参数，自定义的中文方法名称存储在数据库，利用Java 反射机制，在运行时通过类的全路径，方法名称，入参执行调用的方法。

### 调度层

使用调度系统对行为进行调度，会判断在哪一个步骤，进行处理

### 业务保护层
1. 监控埋点，信息采集
2. 多维度业务监控 - 卡单- 还款失败- 跳期校验
3. 限流 1.数据上报 2. 统计，阈值设定   3. 熔断策略    4. 故障恢复------探针
4. 线程池隔离
----------可靠性

5. 数据一致性-对账系统

卡贷（借呗）撮合机构，风控，信贷核心，资金匹配，资金接入

技术架构：
自研 service mesh  side car :把微服务治理的功能收拢到 side car, 增加网络负担  代理
dubbo : 嵌入到服务中

1. 方案制定，多资金方接入，流程的高相似性————————》执行引擎，重复代码少，流程统一，快速开发。提供
2. 运营难点，单一数据源造成的交叉影响，线程池打满，触发 service mesh  过载保护。 ---》 没有保护机制，线程池隔离：单独处理各接口
 FLUX 线程池模型  线程池数量配置 - IO密集型  CPU * 2 与 处理任务最小值
3. 分布式锁，调研分布式锁的实现。setnx 续期问题，主节点宕机未同步。  幂等性如何实现
4. 数据库  冷数据 分表  索引创建 垂直切分  还款场景  理赔

解决问题：
1. 提供工作落地 处理器配置信息
2. 提供基于注解的分布式锁
3. 提供基于注解的 服务调用方式

next key lock 产生规则

说说Guava cache的实现思路，清除策略是怎么实现的
  灵感来源于ConcurrentHashMap，支持缓存清除算法和多种引用类型、使用两个队列来实现LRU清楚策略


Arrays.asList Assays.subList



还款试算的场景，上游提前对正常还款批任务处理的场景进行试算缓存，大量的瞬时流量请求到系统中。
1. 优化点上游提前对批扣场景进行分片处理
2. 对还款计划做缓存，一天只查询一次资金方接口。

如果用削峰加回调的模式可能会出现什么问题：
1. 消息丢失
2. 失败策略怎么处理，如果一直是失败怎么处理，设置封装成一个dto后设置一个ttl ，再放到告警队列中告警处理。
3. mq 的消费策略是怎么样的，是不是先进先出，如果不是有可能存在一直没有被消费的消息
4. 如何判断什么时候可以去消费mq, 当设置有最大值时不去消费渠道的消息
5. 跨天消息不能消费，十一点半清空队列
6  把两个流量入口合并成一个并封装其请求设置ttl ,每次调用后-1，为0时放到告警队列。


处理器落地
1. 获取所有的类
2. 遍历类中的方法
3. 把带有自定义注解的方法添加到map中
### 慢查询的优化：
1. 建表以后的新加的时间字段没有建索引，少数资方使用到，导致慢查询
2. channel 索引删除与 createTime 做联合索引
3. 区分度高的 status 与 proc 联合索引，运营系统需要对正在处理中的订单进行监控。

资金接入组，负责开发维护整个公司的资金端，为公司的资产提供匹配到的资金。
背景：我们组需要为资产和匹配好的资金方做流程对接
难点：不同资金方的对接流程和传输报文字段加解密方式都各不相同。各自开发资金对接系统时间成本大。
如何解决：抽象对接流程，标准化数据源。

解决后：快速的配置化流程两到三天开发完成一个资金方的对接流程。

线程池问题：
是共用了一个静态线程池，父线程等待子线程执行完，子线程又给父线程分配了任务，循环等待死锁了

```
 /**
     * lua 脚本原文
     */
    private static final String LUA_SCRIPT =
                    "local maxRequestKey = KEYS[1] " +
                    "local curRequestKey = KEYS[2] " +
                    "local lowestRequestCount = ARGV[1] " +
                    "local reqId = ARGV[2] " +
                    "local timstampVal = ARGV[3] " +
                    "local result = 0 " +
                    "local maxCount = redis.call('get', maxRequestKey) " +
                    "if maxCount " +
                    "then " +
                    "    local curCount = redis.call('hkeys', curRequestKey) " +
                    "    if(tonumber(maxCount) >= tonumber(lowestRequestCount) and #curCount >= tonumber(maxCount)) " +
                    "    then " +
                    "        return 0 " +
                    "    end " +
                    "    redis.call('hset', curRequestKey, reqId, timstampVal) " +
                    "    return 1 " +
                    "else " +
                    "    redis.call('hset', curRequestKey, reqId, timstampVal) " +
                    "    return 1 " +
                    "end ";
    String maxRequestKey = genRedisKey(channelCode, interfaceName, MAX_REQUEST_SUFFIX);
    String currentRequestKey = genRedisKey(channelCode, interfaceName, CURRENT_REQUEST_SUFFIX);
    
    List<String> keys = Lists.newArrayList(maxRequestKey, currentRequestKey);               
    Long code = this.runLua(LUA_SCRIPT, Long.class, keys, Long.toString(lowestRequestCount), reqId, Long.toString(System.currentTimeMillis() / 1000));

```