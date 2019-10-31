# Spring实战

## Bean的装配

### 自动装配

**当bean都是自己项目里的时候，可以在bean上面添加注解，来实现装配**

@Component

创建可被发现的bean

若不想用spring的注解，还可以用`javax.inject`包里的@Named，也可以将当前bean声明为可被发现



@ComponentScan 

* 不带任何参数，默认是扫描这个带有这个注解的类所在的包

* 使用`basePackages` ，传入字符串参数（可以是多个）,字符串是包的路径

  ```java
  @Configuration
  @ComponentScan(basePackages = {"com.dao","com.io"})
  public class Config{}
  ```

* 使用`baseClasses`，传入类

  可以扫描指定类所在的包

   \>\> 可以在一个包中用一个空的标记接口

  ```java
  @Configuration
  @ComponentScan(baseClasses = {Dao.class,IO.class})
  public class Config{}
  ```

若原先使用`basePackages` ，在重构时，包名路径则可能会出错（因为是字符串），所以使用`baseClasses`可以保持对重构的友好支持





@Autowired

可注解在方法上，会尝试去根据形参，找到合适的bean，形参是多个的话，则spring会尽力去满足所有的依赖

```java
public class JayCDPlayer{
	private CD cd;
    private Food food;
	@Autowired
    public JayCDPlayer(CD cd,Food food){
        this.cd = cd;
        this.food = food;
    }
}
```



|                    | Spring注解 | Java原生注解 |
| ------------------ | ---------- | ------------ |
| 声明该bean可被发现 | @Component | @Named       |
| 依赖注入           | @Autowired | @Inject      |
|                    |            |              |



### 手动装配

当bean是来自第三方包时，无法在别人的代码里加上@Component等注解，此时需要手动显式装配

显式装配又可分为2种

* JAVA装配

  * 简单bean

    使用@Bean

    定义一个@Configuration的注解类，在注解类中，编写创建bean的方法，并在该方法上加上注解@Bean，此注解会告诉spring该方法返回一个bean实例，并将该bean实例注册到spring容器中 ，默认情况，生成的bean对象的名称，是带@Bean注解的方法的名称

    ```java
    @Configuration
    public class Config{
        @Bean
        public jayCD(){
            return new JayCD();
        }
        @Bean(name = "JayPlayer")
        public player(){
            return new JayCDPlayer();
        }
    }
    ```

  * 依赖注入

    ```java
    @Configuration
    public class Config{
        @Bean
        public CD jayCD(){
            return new JayCD();
        }
        @Bean(name = "JayPlayer")
        public CDPlayer player(){
            return new JayCDPlayer(jayCD());
        }
        @Bean
        public CDPlayer anotherPlayer(){
            return new JJCDPlayer(jayCD());
        }
        //这里bPlayer直接请求一个CD对象作为参数
        @Bean 
        public bPlayer(CD cd){
            //spring会自动查找上下文中的bean实例
            return new JayCDPlayer(cd);
        }
        /***
        由于jayCD()方法上有@Bean注解
        所以Spring会拦截第9行代码中的jayCD()的方法调用，
        确保直接返回该方法所创建的bean对象，而不是每次都对
        该方法进行实际调用
        ***/
    }
    ```

    

* XML装配

  * 构造器注入

    ```xml
    <bean id="cd" class="sound.JayCD"/>
        <bean id="food" class="niceFood.Sandwitch"/>
        <bean class="sound.JayCDPlayer">
            <constructor-arg ref="cd"/>
            <constructor-arg ref="food"/>
            <!-- ref表示是引用，value表示字面量 -->
        </bean>
    ```

    
    * c命名空间（c for constructor）

    ```xml
    <!-- 在beans标签内添加
    	xmlns:c="http://www.springframework.org/schema/c" 
    -->
    <bean id="cd" class="sound.JayCD"/>
        <bean id="food" class="niceFood.Sandwitch"/>
        <bean class="sound.JayCDPlayer" c:cd-ref="cd" c:food-ref="food"/>
    
    <!-- cd-ref表明构造函数参数中有一个名为cd的变量，而cd-ref告诉spring正在装配的是一个引用，也可以用形参的位置来进行装配 -->
    <bean class="sound.JayCDPlayer" c:_0-ref="cd" c:_1-ref="food"/>
    ```

  然而，使用`<constructor-arg>` 可以用来装配集合（使用`<list> <set>`等标签），而c命名空间则不行。`<constructor-arg>`是放在了`<bean>`标签的中间，是其子标签，而c命名空间放在了`<bean>`标签的内部，是其属性

  

  * 属性注入

    ```xml
    <bean id="food" class="com.food.Sandwitch">
    	<property name="price" value="10"/>
        <property name="merchant" ref="_7-11"/>
    </bean>
    <bean id="_7-11" class="com.merchant.Seven11"/>
    ```

    * p命名空间（p for property）

      p命名空间遵循的参数命名规范和c命名空间类似

      ```xml
      <!-- 头部beans标签里加上一行
       xmlns:p="http://www.springframework.org/schema/p" 
      -->
      <bean id="food" class="com.food.Sandwitch" p:price="10" p:merchant-ref="_7-11"/>
      <bean id="_7-11" class="com.merchant.Seven11"/>
      ```

      

spring的依赖注入就2种：**构造器注入**，**属性注入**（setter注入）

spring为`<constructor-arg>` 元素提供了c命名空间替代方案

为`<property>` 元素提供了p命名空间替代方案



集合依赖，可以用`<constructor-arg>`或`<property>`来完成注入，其余的依赖，可以直接用c命名空间，或p命名空间来注入。



然而，可以使用spring的util命名空间，来使得p和c命名空间装配集合（util命名空间，提供了list，set，map，properties等）

```xml
<!-- 需要在beans标签内添加一行
xmlns:util="http://www.springframework.org/schema/util"
-->

<util:list id="list">
	<value>Sandwitch</value>
    <value>Hotdog</value>
</util:list>

<bean id="foodStore" class="com.food.store" p:foods-ref="list"/>
```



**强依赖使用构造器注入，可选依赖用属性注入**



手动装配时，可混合使用JAVA Config和XML配置

* JAVA Config中如何使用XML配置中的bean？

  可使用@Import来拆分JAVA Config类，使用@ImportResource来导入XML的配置

```java
@Configuration
@ImportResource("classpath:spring-demo.xml")
public class Config{
    @Bean
    public CDPlayer player(CD cd){
        return new JayCDPlayer(cd);
    }
}
```

* XML配置中如何使用JAVA Config的bean？

  可以使用`<import>`标签来拆分XML文件，使用`<bean>`标签来导入JAVA Config配置类，这样就可以在XML中使用JAVA Config中的bean对象了

  ```xml
  <bean class="com.food.FoodConfig"/>
  ```

更通常的，可以使用一个更高层次的配置文件，其中不声明任何bean，只负责组合JAVA Config和XML配置

```xml
<bean class="com.config.GlobalConfig"/>
<import resource="spring-demo.xml"/>
```



## 多环境配置

* JAVA Config中：使用@Profile，作用于Config类或者某个具体的@Bean方法皆可

* XML中：在`<beans>`标签中添加属性 `profile`

  ```xml
  <beans
          profile="dev"
          xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="
      http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context-4.0.xsd
      http://www.springframework.org/schema/aop
      http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
      ">
  </beans>
  ```



可以在根元素`<beans>`中嵌套`<beans>`元素

```xml
<beans
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
    ">
    <beans profile="dev">
        <bean class="com.food.Food" c:name="devFood"/>
    </beans>
    <beans profile="prod">
        <bean class="com.food.Food" c:name="prodFood"/>
    </beans>
    
    
</beans>
```
