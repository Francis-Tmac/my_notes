## @SpringBootApplication

### @SpringBootConfiguration
- 等价于一个 @Configuration, 本质上是 @Component 也是一个组件

### EnableAutoConfiguration
- 打开 spring boot 自动配置机制，

ImportBeanDefinitionRegistrar

### ComponentScan
允许程序自动扫描包，扫描当前包及其子包下标注了 @Component,
 @Controller, @Service


com.xiaoying.investlog.aop.AopLogService