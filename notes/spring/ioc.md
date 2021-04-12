### 容器
管理一个对象的生到死，创建到销毁的过程。

ioc-优点
1. bean 能够实现集中管理，实现类的可配置和易管理
2. 降低类与类之间的耦合度

###
- 循环依赖
- 事务
- 生命周期
- Ioc
- Aop
- 设计模式
- 传播特性

### 类怎么变成一个bean 
#### 实现
- xml 拿到class 属性，拿到构造器，构造一个实例对象 - ClassPathXmlApplicationContext
- 注解 AnnotationConfigApplicationContext
- javaConfig


一：概念态 ————> 二：定义态（BeanDefinition）------> 三：二，三级缓存，早起暴露bean，循环依赖才能体现此步骤的作用 -------> 四：singletonObjects 最终在应用中使用的bean

三：实例化，依赖注入DI 通过AutoWired 属性注入的，属性设置为空


#### beanDefinition
- scope
- className
- lazyInit
- initMethodName
- beanClass
- dependsOn

### ApplicationContext 和 BeanFactory 有什么区别
- BeanFactory 属于顶层接口，工厂，只生产Bean
- 

### 如何解决循环依赖
三级缓存 三个Map
可以用二级缓存实现
三级缓存，责任明确 性能更好，第三级缓存 是函数接口，主要用来解决aop

BeanFactoryPostProcessor -----> 修改 Bean定义
BeanDefinitionRegistryPostProcessor -----> 注册Bean定义

spring 集成mybatis 时，BeanDefinitionRegistryPostProcessor 
mapper 是接口，ApplicationContext 扫描时会排除接口和抽象类


实例化 ——---> 依赖注入DI ------>初始化 
BeanPostProcessor bean后置处理器
初始化后做 aop 后置处理
aop jdk  CGLIB

FactoryBean 和 BeanFactory 的区别


DefaultListableBeanFactory

AbstractApplicationContext 
调用bean工厂后置处理器。1. 会将class 扫描成bean 定义 2. 调用bean工厂的后置处理器
1. invokeBeanFactoryPostProcessors(beanFactory)

实例化单例bean
finishBeanFactoryInitialization(beanFactory)

创建bean
return createBean(beanName, mbd, args);

解决循环依赖
getSingleton -----> beforeSingletonCreation

1. 实例化一个ApplicationContext 对象
2. 调用bean 工厂后置处理器完成扫描
3. 循环解析扫描出来的类信息
4. 实例化一个BeanDefinition 对象存储解析出来的信息
5. 实例化好的 beanDefinition 对象put 到 beanDefinitionMap 中存储起来，以便后面实例化bean
6. 再次调用bean 工厂后置处理器
7. 国际化等操作，实例化一个bean 重要的是调用 finishBeanFactoryInitialization 方法来实例化单例的bean ，实例化之前spring要做验证，需要遍历所有扫描出来的类，
            一次判断这个bean 是否 lazy，是否 protoType，是否 abstract等等。
8. 如果验证完成spring 在实例化一个bean 之前 需要推断构造方法，因为spring 实例化对象是通过构造方法反射，故需要知道用的哪个构造方法。
9. 推断完成构造方法反射实例化一个对象；注意这了是对象，对象，对象；这时候对象已经实例化出来了，但并不是一个完整的bean，最简单的体现就是这时候实例化出来的对象属性是没有注入，所有不是一个完整的bean；
10. spring 处理合并后的 beanDefinition
11. 判断是否需要完成属性注入
12. 如果需要完成属性注入，则开始注入属性
13. 判断bean 的类型回调Aware 接口
14. 调用生命周期回调方法
15. 如果需要代理则完成代理
16. put 到单例池---bean完成---存在spring 容器中


DI：依赖注入，把想要注入对象所依赖的bean，自动注入进去。正是通过依赖注入使得 IOC中的bean 解耦，在依赖注入的时候只需要让他依赖一个接口，
而不是依赖具体的实现，
aop: 面向界面编程，遇到很多重复性的代码，比如日志，事务。把共有的代码抽取出来，然后切入到想要，减少冗余代码提高代码的复用性。



谈一下对spring的理解：
- spring是一个框架，在现有的系统中所有的框架
spring中的IOC 容器用来存储所有的bean 对象，它管理了这些bean从实例化 初始化 到销毁的整个生命周期。
spring 在讲类初始化为一个bean 对象的过程中，需要先将原对象的信息装换为一个 beanDefinition ,完成了整个beanDefinition的解析和加载过程后。
通过beanDefinition 的信息用反射方式对bean进行创建操作，


