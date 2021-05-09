# 原理
## @EnableAspectJAutoProxy
- @Import(AspectJAutoProxyRegistrar.class) 给容器中导入 `AspectJAutoProxyRegistrar`
    - 利用`AspectJAutoProxyRegistrar` 自动以给容器中注册 bean；

`AnnotationAwareAspectJAutoProxyCreator`
    -> `AspectJAwareAdvisorAutoProxyCreator`
        -> `AbstractAdvisorAutoProxyCreator`
            -> `AbstractAutoProxyCreator`
                -> `AbstractAutoProxyCreator implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware`
                
