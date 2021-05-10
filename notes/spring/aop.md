# 原理
## @EnableAspectJAutoProxy
- @Import(AspectJAutoProxyRegistrar.class) 给容器中导入 `AspectJAutoProxyRegistrar`
    - 利用`AspectJAutoProxyRegistrar` 自动以给容器中注册 bean；

`AnnotationAwareAspectJAutoProxyCreator`
    -> `AspectJAwareAdvisorAutoProxyCreator`
        -> `AbstractAdvisorAutoProxyCreator`
            -> `AbstractAutoProxyCreator`
                -> `AbstractAutoProxyCreator implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware`
                
`AbstractAutoProxyCreator#setBeanFactory()`
`AbstractAutoProxyCreator.后置处理器逻辑`

`AbstractAdvisorAutoProxyCreator#setBeanFactory()->initBeanFactory()`

`AnnotationAwareAspectJAutoProxyCreator#initBeanFactory()`


``` java
public class AOPTest {  
    @Test  
  public void test_1(){  
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(  
                MainConfigOfAOP.class);  
  MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);  
  mathCalculator.div(4,2);  
  }  
}
```

- 传入配置类，创建IOC 容器  
- 注册配置类，调用refresh() 刷新容器  
- `registerBeanPostProcessors(beanFactory);` 注册bean 的后置处理器拦截bean的创建
    - 现获取IOC 容器已经定义的需要创建对象的所有 `BeanPostProcessor`
    - 给容器中加别的  `BeanPostProcessor`
    - 分离实现 `PriorityOrdered`和 `Ordered` 接口的 `BeanPostProcessor`
    - 优先注册实现 `PriorityOrdered` 接口的`BeanPostProcessor`
    - 在给注册实现 `Ordered` 接口的`BeanPostProcessor`
    - 最后注册普通的`BeanPostProcessor`
    - 注册 `BeanPostProcessor`，实际上就是创建 `BeanPostProcessor` 对象，保存在容器中。
        - 创建 internalAutoProxyCreator的 BeanPostProcessor
        - 创建bean的实例
        - populateBean: 给bean 的各种属性赋值
        - initializeBean: 初始化bean 
            - invokeAwareMethods(): 处理Aware接口的方法回调
- 