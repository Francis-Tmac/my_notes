## @SpringBootApplication

### @SpringBootConfiguration
- 等价于一个 @Configuration, 本质上是 @Component 也是一个组件

### EnableAutoConfiguration
- 打开 spring boot 自动配置机制，

ImportBeanDefinitionRegistrar

```` 
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

}

public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

    @Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>.判断自动装配开关是否打开
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
        //<2>.获取所有需要装配的bean
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
				autoConfigurationMetadata, annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}


    protected AutoConfigurationEntry getAutoConfigurationEntry(
    			AutoConfigurationMetadata autoConfigurationMetadata,
    			AnnotationMetadata annotationMetadata) {
            // 1. 判断自动装配开关是否打开
    		if (!isEnabled(annotationMetadata)) {
    			return EMPTY_ENTRY;
    		}
            // 2. 用于获取EnableAutoConfiguration注解中的 exclude 和 excludeName。
    		AnnotationAttributes attributes = getAttributes(annotationMetadata);
            // 3. 获取需要自动装配的所有配置类，读取META-INF/spring.factories
            // 所有 Spring Boot Starter 下的META-INF/spring.factories都会被读取到。
    		List<String> configurations = getCandidateConfigurations(annotationMetadata,
    				attributes);
    		configurations = removeDuplicates(configurations);
    		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    		checkExcludedClasses(configurations, exclusions);
    		configurations.removeAll(exclusions);
            // 4. 经历了一遍筛选，@ConditionalOnXXX 中的所有条件都满足，该类才会生效。
            // configurations 的值变小了。 实测 150 ——> 63
    		configurations = filter(configurations, autoConfigurationMetadata);
    		fireAutoConfigurationImportEvents(configurations, exclusions);
    		return new AutoConfigurationEntry(configurations, exclusions);
    	}

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
    /**
     * 该方法主要用于获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中。
     **/
    String[] selectImports(AnnotationMetadata var1);
}
````
- Spring Boot 通过@EnableAutoConfiguration开启自动装配，通过 SpringFactoriesLoader 最终加载META-INF/spring.factories中的自动配置类实现自动装配，自动配置类其实就是通过@Conditional按需加载的配置类，想要其生效必须引入spring-boot-starter-xxx包实现起步依赖

### ComponentScan
允许程序自动扫描包，扫描当前包及其子包下标注了 @Component,
 @Controller, @Service

@ComponentScan + @Component
@Bean + @Configuration
@Import + 接口

### AnnotationConfigUtils#registerAnnotationConfigProcessors
```
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, @Nullable Object source) {

		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
		if (beanFactory != null) {
			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
				beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
			}
			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
			}
		}

		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
        // 实现import + 接口注入
		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        // 实现 @Autowire 注入
		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        // 实现 @Resource 注入
		// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition();
			try {
				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
						AnnotationConfigUtils.class.getClassLoader()));
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
			}
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
		}

		return beanDefs;
	}
```







