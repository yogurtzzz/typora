spring 的bean的生命周期

1. 实例化

2. 依赖注入

3. BeanNameAware

4. BeanFactoryAware

5. ApplicationContextAware

6. **BeanPostProcessor的postProcessBeforeInitialization**

   注意：若一个bean，实现了BeanPostProcessor接口，并且该bean被纳入spring容器，那么这个bean中的`postProcessBeforeInitialization`和`postProcessAfterInitialization`会对所有bean的创建进行拦截，统一进行前置后置处理

7. `@PostConstruct`方法
8. InitializingBean的afterPropertiesSet

9. `<bean>标签中的init-method` 

10. **BeanPostProcessort的postProcessAfterInitialization**

11. `@PreDestroy`方法
12. `<bean>标签中的destroy-method`
13. DisposableBean的destroy方法