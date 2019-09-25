```java
//某个Configuration配置类中
@Configuration
@PropertyResource("env.properties")
public class Config{
	@Bean
	@Conditional(Yogurt.class)
	public Food food(){
    	return new Hamburger();
	}
}
```



```java
//Yogurt.class
public class Yogurt implements Condition{
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String yogurt = environment.getProperty("yogurt");
        return "on".equals(yogurt);
        //当环境变量中yogurt的值是on时，满足条件，创建Hamburger这个bean
    }
}
```



```properties
#env.properties
yogurt=on
```

