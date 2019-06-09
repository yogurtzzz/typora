# Spring笔记

## Spring的特点

* `IoC`和`DI` 功能，将对象的创建和管理交由spring框架控制，解耦，使开发人员更专注于业务代码
* `AOP`支持，spring提供AOP功能，方便进行**面向切面编程**
* 声明式事务支持，解耦，将事务管理交由spring控制，提高开发效率
* 对其他优秀框架有很好的兼容，方便集成开发，spring提供了对诸如struts,hibernate,Hessian,Quartz，JUnit等优秀框架的直接支持
* 封装了一些JAVA EE API，降低使用难度，spring对诸如JDBC，JAVAMail，远程调用等功能进行了一层封装，降低了开发人员对API的使用难度
* 其源码是经典的学习范例。spring的源码设计精妙优雅，无疑是java技术的最佳实践。学习和研究spring源码将对自己提高java技术水平有很大的帮助



## Spring核心基础

* **IoC** : Inversion of Control  控制反转，对象的创建权力交由spring框架
* **DI** : Dependency Injection  依赖注入，将对象的属性动态进行注入
* **AOP** : Aspect Oriented Programming 面向切面编程，在不修改目标对象代码的前提下，增强对象的功能，如日志记录，事务操作等。
* **Spring容器** : 即IoC容器，一个BeanFactory





### XML开发方式

#### IOC

在xml文件中通过bean标签，完成IoC的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:task="http://www.springframework.org/schema/task"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
			http://www.springframework.org/schema/context
			http://www.springframework.org/schema/context/spring-context-3.0.xsd
            http://www.springframework.org/schema/aop
			http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
            http://www.springframework.org/schema/tx
			http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
			http://www.springframework.org/schema/util
			http://www.springframework.org/schema/util/spring-util-3.0.xsd" >
    <bean id="propertyConfigurerWeb"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="fileEncoding" value="UTF-8"/> 
		<property name="localOverride" value="true" />
		<property name="placeholderPrefix" value="${" />
		<property name="placeholderSuffix" value="}" />
		<property name="locations">
			<list>
				<value>classpath:applicationConfig.properties</value>
			</list>
		</property>
	</bean>
</beans>
```

##### **bean标签属性**

* id : 给对象在容器中提供一个唯一标识，用于获取对象

* class : 指定类的全限定名，用于反射创建对象，默认情况下调用**无参的构造函数**

* init-method : 指定类的初始化方法名称

* destroy-method : 指定类的销毁方法名称，如DataSource的配置中一般需要指定destroy-method="close"

* scope : 指定对象的作用范围（生命周期）

  * singleton : **默认值**，单例（整个容器中只有一个对象），其生命周期如下：
    * 对象出生：应用加载，创建容器时，对象被创建
    * 对象活着：容器在，对象就在
    * 对象死亡：卸载应用，销毁容器时，对象被销毁

  * prototype：多例，每次访问对象时，都会重新创建对象实例，生命周期如下：
    * 对象出生：当使用对象时，创建新的对象实例
    * 对象活着：只要对象在使用中，就一直活着
    * 对象死亡：对象长时间不用时，会被java的垃圾回收器回收
  * request：将Spring创建的bean对象存到request域中
  * session：将Spring创建的bean对象存到session域中
  * global session：WEB项目中，应用在Portlet环境，若无Portlet环境，那么该global session相当于session

##### **bean实例化的方式**

* **使用默认的无参构造函数 （重点）**

  默认情况下，会根据无参数构造函数来创建类对象，若bean中没有默认的无参构造函数，将会创建失败

  ```xml
  <bean id="myService" class="com.yogurt.service.MyServiceImpl"/>
  ```

* **静态工厂（了解）**

  ```xml
  <bean id="myService" class="com.yogurt.service.StaticFactory" factory-method="createService"/>
  ```

  ```java
  package com.yogurt.service;
  public class StaticFactory{
      public static MyService createService(){
          return new MyServiceImpl();
      }
  }
  ```

* **实例工厂（了解）**

  ```xml
  <!-- 即先创建一个factory bean，然后使用该bean里的方法来创建对象-->
  <bean id="instanceFactory" class="com.yogurt.service.InstanceFactory"/>
  <bean id="myService" factory-bean="instanceFactory" factory-method="createService"/>
  ```

  ```java
  package com.yogurt.service;
  public class InstanceFactory{
      public MyService createService(){
          return new MyServiceImpl();
      }
  }
  ```

#### DI

什么是依赖？

> 依赖指的是bean实例中的属性
>
> 依赖（属性）分为：**简单类型（8种基本类型和String）**  ， **POJO类型**，**集合数组类型**



什么是依赖注入（Dependency Injection）

> 是Spring框架IoC的具体实现



##### 依赖注入的方式

* **构造函数注入**

  即使用类的构造函数，给成员变量赋值。注：赋值的操作是通过配置的方式，让Spring来完成的

  ```java
  public class MyServiceImpl implements MyService{
      private String name;
      private int id;
      private double price;
      
      public MyServiceImpl(String name,int id,double price){
          this.name = name;
          this.id = id;
          this.price = price;
      }
      
      @Override
      public void doService(){
          System.out.println("do service now");
      }
  }
  ```

  

  

  ```xml
  <bean id="myService" class="com.yogurt.service.MyServiceImpl">
      <constructor-arg name="name" value="camera"></constructor-arg>
      <constructor-arg name="id" value="101"></constructor-arg>
      <constructor-arg name="price" value="3000"></constructor-arg>
  </bean>
  
  <!--
  使用构造函数进行依赖注入，要求：类中需要提供一个对应参数列表的构造函数
  
  涉及的标签：constructor-arg    该标签有如下属性
  	* index: 指定参数在构造函数参数列表中的索引位置
  	* name: 指定参数在构造函数参数列表中的名称
  	* value: 能对8种基本类型和String类型进行赋值
  	* ref: 能对POJO类型进行赋值，但该bean必须在容器中配置过
  ```

* **set方法注入（重点）**

  * 手动装配方式（XML）

    > * 需配置bean标签的子标签property
    > * 需要在bean类中指定setter方法

    ```xml
    <bean id="person" class="com.yogurt.po.Person">
    	<property name="name" value="大黄"></property>
        <property name="age" value="23"></property>
        <property name="detail" ref="detail"></property>
        <property name="friends">
        	<list>
            	<ref bean="friend1"/>
                <ref bean="friend2"/>
            </list>
        </property>
    </bean>
    <bean id="friend1" class="com.yogurt.po.Person">
    	<property name="name" value="崔"></property>
    </bean>
    <bean id="friend2" class="com.yogurt.po.Person">
    	<property name="name" value="孔"></property>
    </bean>
    <bean id="detail" class="com.yogurt.po.Detail"></bean>
    ```

    ```java
    package com.yogurt.po;
    /** 
     * @ClassName: Person 
     * @Description: TODO 
     * @author: yogurtbee 
     * @date: 2019-5-6
     * 
     */
    public class Person {
        private String name;
        private int age;
        private Person[] friends;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public int getAge() {
            return age;
        }
        public void setAge(int age) {
            this.age = age;
        }
        public Person[] getFriends() {
            return friends;
        }
        public void setFriends(Person[] friends) {
            this.friends = friends;
        }
    }
    ```

    * 注入不同类型的属性

      * 简单类型 （value）

        ```xml
        <!--8种基本类型和String类型，皆通过此种方式 -->
        <bean id="person" class="com.yogurt.po.Person">
        	<property name="name" value="yogurt"></property>
            <property name="age" value="23"></property>
        </bean>
        ```

      * 引用类型（ref）

        ```xml
        <!--POJO属性通过 ref 来注入 -->
        <bean id="person" class="com.yogurt.po.Person">
        	<property name="name" value="yogurt"></property>
            <property name="age" value="23"></property>
            <property name="mother" ref="yogurtMom"></property>
        </bean>
        
        <bean id="yogurtMom" class="com.yogurt.po.Person"></bean>
        ```

        

      * 集合类型

        * 数组或List

          ```xml
          <bean id="person" class="com.yogurt.po.Person">
              <property name="friends">
              	<list>
            <!--如果集合内是简单类型或String,使用value子标签，若是POJO，使用ref-->
                  	<ref bean="friend1"/>
                      <ref bean="friend2"/>
                  </list>
              </property>
          </bean>
          
          <bean id="friend1" class="com.yogurt.po.Person"/>
          <bean id="friend2" class="com.yogurt.po.Person"/>
          
          ```

          

        * Set

          ```xml
          <bean id="person" class="com.yogurt.po.Person">
              <property name="words">
              	<set>
            <!--如果集合内是简单类型或String,使用value子标签，若是POJO，使用ref-->
                      <value>哈哈</value>
                      <value>嘿嘿</value>
                  </set>
              </property>
          </bean>
          ```

          

        * Map

          ```xml
          <bean id="person" class="com.yogurt.po.Person">
              <property name="homes">
                  <map>
                  	<entry key="江油" value="明月新城"/>
                      <entry key="广州" value="南悦花苑"/>
                  </map>
              </property>
          </bean>
          
          <!--
          若map中的value是POJO类型的话，如Map<String,Person> map
          则用<ref>子标签进行注入，如下
          -->  
          ```

          

          ```xml
          <bean id="seller" class="com.yogurt.po.Seller">
                          <property name="shopName" value="大黄的小店"/>
                          <property name="products">
                              <map>
                                  <entry key="Computer">
                                      <ref bean="pc"/>
                                  </entry>
                                  <entry key="Ps">
                                      <ref bean="ps4"/>
                                  </entry>
                              </map>
                          </property>
              			<property name="props">
                              <props>
                                  <prop key="yes">haha</prop>
                                  <prop key="no">dxixi</prop>
                                  <prop key="fu">f**k</prop>
                              </props>
                          </property>
                      </bean>
                      
                      <bean id="pc" class="com.yogurt.po.Product">
                          <property name="name" value="外星人X5"/>
                          <property name="price" value="10000"/>
                      </bean>
                      
                      <bean id="ps4" class="com.yogurt.po.Product">
                          <property name="name" value="PlayStation4"/>
                          <property name="price" value="30000"/>
                      </bean>
          ```

          

          

          ```java
          package com.yogurt.po;
          import java.util.Map;
          /** 
           * @ClassName: Seller 
           * @Description: TODO 
           * @author: yogurtbee 
           * @date: 2019-5-7
           * 
           */
          public class Seller {
              private int id;
              private String shopName;
              private Map<String, Product> products;
              private Properties props;
          	//这里省略了set/get函数
          }
          ```

          ```java
          package com.yogurt.po;
          /** 
           * @ClassName: Product 
           * @Description: TODO 
           * @author: yogurtbee 
           * @date: 2019-5-7
           */
          public class Product {
              private int id;
              private String name;
              private String price;
          	//这里省略了set/get函数
          }
          ```

          

        

        

        * Properties

          ```xml
          <bean id="person" class="com.yogurt.po.Person">
              <property name="detail">
                  <props>
                  	<prop key="uname">root</prop>
                      <prop key="password">123</prop>
                  </props>
              </property>
          </bean>
          ```

          测试代码

          ```java
          public static void main(String[] args) throws {
                  ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-demo.xml");
                  Seller seller = (Seller) ctx.getBean("seller");
                  System.out.println("shopName = " + seller.getShopName());
                  Properties properties = seller.getProps();
                  properties.list(System.out);
          }
          ```

          输出

          ![1558270298123](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558270298123.png)

          

          

  * 自动装配方式（注解）

    > * @Autowired
    >   * 在spring容器中，根据bean的类型(byType)查找并获取实例
    >   * 将找到的实例对象，装配给另一个实例的属性值
    >   * 注：一个java类型在同一个spring容器中，只能有一个实例
    > * @Resource
    >   * 在spring容器中，根据bean的名称(byName)查找并获取实例
    >   * 将找到的实例对象，装配给另一个实例的属性值

* p命名空间注入（本质上还是调用set方法）

  * 步骤一：需要先在xml中引入p命名空间

    > 在schema的命名空间中加入： xmlns:p="http://www.springframework.org/schema/p"

  * 步骤二：使用p命名空间的语法

    > p:属性名 = ""
    >
    > p:属性名-ref = ""

  * 步骤三：实践

    ```xml
    <bean id="person" class="com.yogurt.po.Person" p:name="大黄" p:detail-ref="detail"></bean>
    <bean id="detail" class="com.yogurt.po.Detail"></bean>
    ```

    

### 混合方式（XML+注解 ）

#### IOC注解

主要用到4个注解：@Component以及其余3个衍生注解 @Controller，@Service，@Repository

即创建对象到spring容器中，相当于xml中的

```xml
<bean id="xx" class="xxx"/>
```



* 第一步：在spring的配置文件中，配置context:component-scan标签

  ```xml
  <!-- spring-demo.xml -->
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context-3.0.xsd">
              <context:component-scan base-package="com.yogurt.annotation"/>
  </beans>
                  
  ```

  

* 第二步：在类上面加上IoC注解  @Component，或者它的衍生注解@Controller，@Service，@Repository

  ```java
  package com.yogurt.annotation;
  import org.springframework.stereotype.Component;
  /** 
   * @ClassName: AnnotationPerson 
   * @Description: TODO 
   * @author: yogurtbee 
   * @date: 2019-5-7 
   */
  //不指定value属性的话，bean的id默认是当前类的类名，首字母小写
  //若注解中有且仅有一个属性要赋值，且该属性是value，那么可以省略value
  //即，下一行代码等价于  @Component("aPerson")
  @Component(value="aPerson")
  public class AnnotationPerson {
      private int id;
      private String name;
      private int age;
      public AnnotationPerson(){
          this.id = 1;
          this.name = "anonymous";
          this.age = 1;
      }
  }
  ```

* 第三步：测试

  ```java
  public static void main(String[] args){
          ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-demo.xml"); 
          AnnotationPerson person = (AnnotationPerson) ctx.getBean("aPerson");
          System.out.println("name = " + person.getName());
      }
  ```

* Controller，Service，Repository注解

  这三个注解都是针对@Component的衍生注解，他们的作用和属性都是一模一样的。唯一的区别仅仅在于，这三者提供了更加明确的语义化

  > @Controller 一般用于表现层
  >
  > @Service 一般用于业务层
  >
  > @Repository  一般用于持久层

#### DI注解

相当于xml中的

```xml
<property name="xx" value="xx"></property>
<property name="xx" ref="xx"></property>
```

* **@Autowired**

  * 默认按类型装配（byType）

  * 是Spring自带的

  * 由AutowiredAnnotationBeanPostProcessor类实现

  * 默认情况下要求依赖的对象**必须存在**，若允许null，可设置它的required属性为false，如`@Autowired(required=false)`

  * 若想按名称装配（byName），可以和@Qualifier注解连用，如

    ```java
    @Autowired
    @Qualifier("aPerson")
    private Person person;
    ```

* **@Qualifier**

  * 在自动按类型注入的基础上，再按照Bean的id来注入
  * 在给类的属性注入时，必须和**@Autowired**一起使用
  * 在给方法的参数注入时，可以单独使用

* **@Resource**

  * 默认按名称装配（byName），可通过name属性指定bean的id，若未指定name属性，默认取字段名，按照名称查找，当找不到与名称匹配的bean时，再按类型装配
  * 属于J2EE JSR250规范的实现
  * 需要注意，当name属性指定时，则只会按照名称进行装配

推荐使用@Resource，因为它是属于J2EE的，减少了与Spring的耦合

* **@Inject**

  * 是根据类型进行装配，若要按名称装配，则需要与@Named连用
  * 是JSR330中的规范，需导入javax.inject.Inject
  * 可用在变量，set方法，构造函数上

* **@Value**

  * 给8种基本类型和String类型注入值

  * 可以使用占位符获取properties文件中的值

    ```java
    @Value("${name}")
    private String name;
    ```

    ```properties
    #这里是properties文件
    name=bigYellow
    ```

@Autowired，@Resource，@Inject  三者对比

* @Autowired是Spring的，@Inject是JSR330规范实现的，@Resource是JSR250规范实现的，需导入不同的包
* @Autowired和@Inject用法基本一样，不同的是，@Autowired有一个required属性
* @Autowired和@Inject默认按照类型装配（byType），而@Resource是按名称装配
* 若要按名称装配，@Autowired要和@Qualifier一起使用，@Inject和@Named一起



其他一些注解

* @Scope ： 指定bean的作用范围，相当于

  ```xml
  <bean id="" class="" scope=""/>
  ```

* @PostConstruct

* @PreDestroy

  相当于

  ```xml
  <bean id="" class="" init-method="" destroy-method="" />
  ```

  

### 纯注解方式

关键是去掉spring配置文件中的

```xml
<context:component-scan base-package="com.yogurt.po"></context:component-scan>
```



#### @Configuration

> 相当于spring的xml配置文件
>
> 从Spring 3.0开始， 可以用@Configuration定义配置类，用于替换xml配置文件
>
> 配置类内部包含  被@Bean注解的方法，这些方法会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类扫描，用于构建对象，初始化Spring容器

* 属性：

  value : 用于指定配置类的字节码

* 示例代码

  ```java
  @Configuration
  public class SpringConfiguration{
      //Spring容器初始化时，会调用配置类的无参构造函数
      public SpringConfiguration(){
          System.out.println("初始化Spring容器中...");
      }
  }
  ```



#### @Bean

> 相当于`<bean>`标签
>
> 作用是：注册bean对象，主要用来配置非自定义的bean类，如SqlSessionFactory
>
> @Bean 标注再方法上 （某个返回对象实例的方法）

* 属性

  > name : 给当前@Bean方法返回的对象指定一个名称，即bean的id，若不指定，默认与标注的方法名相同
  >
  > @Bean注解默认的作用域为singleton，可通过@Scope("prototype") 设置为原型作用域

* 示例代码

  ```java
  @Configuration
  public class SpringConfiguration{
      public SpringConfiguration(){
          System.out.println("初始化Spring容器中...");
      }
      @Bean
      @Scope("prototype")
      public SqlSessionFactory myService(){
          SqlSessionFactory sessionFactory = new DefaultSqlSessionFactory();
          sessionFactory.setXXX(xxx);
          return sessionFactory;
      }
  }
  ```

#### @ComponentScan

> 相当`<context:component-scan base-package="xxxx.xxx"/>`
>
> 该注解是加在类上面，一般与@Configuration一起使用

* 属性

  > basePackages : 用于指定扫描的包
  >
  > value : 和basePackages一样

* 示例代码

  Bean类

  ```java
  package com.yogurt.service;
  @Service
  public class MyServiceImpl implements MyService{
      @Override
      public void execute(){
          System.out.println("执行一些操作...");
      }
  }
  ```

  配置类

  ```java
  @Configuration
  @ComponentScan(basePackages="com.yogurt.service")
  public class SpringConfiguration{
      public SpringConfiguration(){
          System.out.println("初始化Spring容器中...");
      }
  }
  ```



#### @PropertySource

> 相当于`<context:property-placeholder>` 标签
>
> 编写在类上，用于加载properties配置文件



* 属性

  > value[] : 用于指定properties文件的路径，若在类路径下，要加上classpath

 

* 示例代码

  ```java
  @Configuration
  @PropertySource("classpath:config.properties")
  public class SpringConfiguration{
      @Value("${name}")
      private String name;
      @Value("${home}")
      private String home;
      //.....
      
      public SpringConfiguration(){
          //扫描到configuration注解的类时，会先调用无参构造函数
          System.out.println("正在初始化Spring容器");
      }
      @Bean(name="annotationPerson")
      public Person createPerson(){
          Person person = new Person();
          person.setName(name);
          person.setHome(home);
          return person;
      }
  }
  ```

  ```properties
  #config.properties
  name=fuck
  home=fuckhome
  ```

  测试代码

  ```java
  public static void main(String[] args){
      ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfiguration.class);
      Person person = ctx.getBean(Person.class);
      System.out.println("name = "+person.getName()+", home = "+person.getHome());
  }
  ```



##### 注：待解决问题

**（已解决，经查实，是为spring版本的问题，使用jdk1.8 + spring 5.1.6 即能成功注入）**

![1557637309991](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557637309991.png)

然而在实践中，发现通过@PropertySource注解，再配合@Value，无法实现注入

![1557637396448](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557637396448.png)

改成如下形式，才能完成注入

```java
@Configuration
@PropertySource("classpath:config.properties")
public class SpringConfiguration{
    @Autowired
    Environment env;
    //通过Environment
    public SpringConfiguration(){
        //扫描到configuration注解的类时，会先调用无参构造函数
        System.out.println("正在初始化Spring容器");
    }
    
    @Bean(name="annotationPerson")
    public Person createPerson(){
        Person person = new Person();
        person.setName(env.getProperty("name"));
        person.setHome(env.getProperty("home"));
        return person;
    }
}

//------------------或者改成如下，用@ImportResource注解

@Configuration
@ImportResource("classpath:config.xml")
public class SpringConfiguration{
    @Value("${name}")
    private String name;
    
    @Value("${home}")
    private String home;
    
    public SpringConfiguration(){
        //扫描到configuration注解的类时，会先调用无参构造函数
        System.out.println("正在初始化Spring容器");
    }
    
    @Bean(name="annotationPerson")
    public Person createPerson(){
        Person person = new Person();
        person.setName(name);
        person.setHome(home);
        return person;
    }
}
```

注意@ImportResource注解后面需要解析xml文件

```xml
<!--config.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-3.0.xsd"> 
    <context:property-placeholder location="classpath:config.properties"/>            
</beans>
```



#### @Imoport

> 相当于Spring配置文件中的`<import>`标签
>
> 用来组合多个配置类，在引入其他配置类时，可以不用再写@Configuration注解

* 属性

  > value : 用来指定其他配置类的字节码文件

* 示例代码

  ```java
  @Configuration
  @ComponentScan(basePackages="com.yogurt.service")
  @Import({MyConfig.class})
  public class SpringConfiguration{
      
  }
  ```

  

  ```java
  @Configuration
  @PropertySource("classpath:myConfig.properties")
  public class MyConfig{
      //...
  }
  ```





#### 创建纯注解方式上下文

* Java应用（AnnotationConfigApplicationContext）

  ```java
  ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfiguration.class);
  MyService myService = ctx.getBean("myService",MyService.class);
  //注意，使用@Configuration需要依赖CGLIB的jar包，需要导入cglib-nodep.jar
  ```



* Web应用（AnnotationConfigWebApplicationContext）

  ```xml
  <web-app>
  	<context-param>
      	<param-name>contextClass</param-name>
          <param-value>
         org.springframework.web.context.support.AnnotationConfiWebApplicationContext
          </param-value>
      </context-param>
      
      <context-param>
      	<param-name>contextConfigLocation</param-name>
          <param-value>com.yogurt.config.SpringConfiguration</param-value>
      </context-param>
      
      <listener>
      	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
  </web-app>
  ```

  

## Spring高级篇

### AOP

#### 基本概念

* 含义：Aspect Oriented Programming，面向切面编程

* 作用：再不修改**目标类**代码的前提下，通过AOP技术，去增强目标类的功能

* 实现：通过【预编译】和【运行期动态代理】实现

  

* **为什么用AOP**

  * 将一些公共代码抽取出来，避免重复编写某些代码，将**业务逻辑**和**系统处理**的代码解耦。如，事务管理，日志记录，权限控制，安全控制，性能统计等



* 相关术语

  * JoinPoint（连接点）

    > 指被拦截到的点。在spring中，连接点指的是某些方法

  * PointCut（切入点）

    > 确认要对哪些JoinPoint进行切入增强

    **举个栗子，JoinPoint就是菜单上的选项，PointCut是你点的菜。即，JoinPoint是咱们感兴趣的，可以去切的方法，而一旦我们选择了哪些方法要被切，那就是PointCut。所以其实根本不用关心JoinPoint。比如咱们有10个方法可以切，而PointCut表达式只匹配了其中2个，那么这10个方法就是JoinPoint，被匹配到的2个方法就是PointCut**

  * Advice（通知/增强）

    > 即切面要完成的功能，拦截到JoinPoint后，需要做的事，分为**前置通知**，**后置通知**，**环绕通知**，**最终通知**，**异常通知**

  * Introduction（引介）

    > 可以在运行期为类动态地添加一些方法或属性

  * Target（目标对象）

    > 需要被增强的对象

  * Weaving（织入）

    > 指对目标对象进行增强，生成新的代理对象的过程

  * Proxy（代理）

    > 一个目标对象被AOP织入增强后，产生一个代理对象

  * Aspect（切面）

    > 切入点+通知，即作用在具体pointCut上的advice

  * Advisor（通知器/顾问）

    > 和Aspect类似

#### AOP之AspectJ

AspectJ是一个Java实现的AOP框架，是目前AOP框架中最成熟，功能最丰富的，能够与Java程序完全兼容

对于织入，一般分为**动态织入** 和 **静态织入**

* **动态织入**：在运行时，动态地将要增强的代码，织入到目标对象中。往往是通过**动态代理技术**来实现的，动态代理一般又分为**基于JDK的动态代理**（基于接口，反射）  和   **基于CGLib的动态代理** （基于继承）

* **静态织入**：在编译期织入，通过AOP框架提供的命令进行编译，在编译阶段就产生AOP代理对象



AspectJ 采用的是**静态织入**，主要是在**编译期织入**，在这期间使用AspectJ的acj编译器（类似于javac） 将aspect类编译成class字节码后，在java目标类编译时进行织入，即先编译aspect类，再编译目标类

![1557637543230](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557637543230.png)



#### AOP之Spring AOP

* Spring AOP是通过**动态代理**实现的，是动态织入



* 基于JDK的动态代理（基于接口）
  * 目标对象必须实现接口
* 基于CGLib的动态代理（基于继承）
  * 目标对象无需实现接口
  * 底层是通过继承目标对象，产生代理子对象的方式，代理子对象中继承了目标对象的方法，并对方法进行了增强

#### AOP的使用（XML方式）

Spring+AspectJ整合使用，Spring已将AspectJ纳入到框架中，且底层实现仍然是动态织入

需要的依赖：

```xml
spring-aop.jar
aopalliance.jar
aspectjlib.jar
aspectj
```

步骤

* 编写目标类

  * 编写接口和实现类

    ```java
    package com.yogurt.aop;
    /** 
     * @ClassName: YogurtService 
     * @Description: TODO 
     * @author: yogurtbee 
     * @date: 2019-5-8
     * 
     */
    public interface YogurtService {
        public String sayHello();
        public String sayByeBye();
    }
    
    /** 
     * @Title:TODO  
     * @Desription:TODO
     * @Company:CSN
     * @ClassName:YogurtServiceImpl.java
     * @Author:yogurtbee
     * @CreateDate:2019-5-8   
     * @UpdateUser:yogurtbee  
     * @Version:0.1 
     *    
     */ 
    ===============================================================================
    package com.yogurt.aop;
    /** 
     * @ClassName: YogurtServiceImpl 
     * @Description: TODO 
     * @author: yogurtbee 
     * @date: 2019-5-8
     * 
     */
    public class YogurtServiceImpl implements YogurtService {
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        @Override
        public String sayHello() {
            System.out.println("Hello from "+this.name);
            return this.name + ",this is Hello return";
        }
        @Override
        public String sayByeBye() {
            System.out.println("Bye bye from yogurt");
            return this.name + ",this is Bye-Bye return";
        }
    
    }
    ```

* 编写通知类（增强类）

  ```java
  /** 
   * @Title:TODO  
   * @Desription:TODO
   * @Company:CSN
   * @ClassName:Advice.java
   * @Author:yogurtbee
   * @CreateDate:2019-5-8   
   * @UpdateUser:yogurtbee  
   * @Version:0.1 
   *    
   */ 
  package com.yogurt.aop;
  import org.aspectj.lang.ProceedingJoinPoint;
  /** 
   * @ClassName: Advice 
   * @Description: TODO 
   * @author: yogurtbee 
   * @date: 2019-5-8
   */
  public class MyAdvice {
      public void log(){
          System.out.println("这里是advice");
      }
      public Object around(ProceedingJoinPoint joinPoint){
          System.out.println("advice 360° 无死角环绕");
          Object returnVal = null;
          try {
              returnVal = joinPoint.proceed();
          } catch (Throwable e) {
              e.printStackTrace();
          }
          System.out.println("advice 360° 无死角环绕");
          return returnVal;
      }
      
      public void afterReturn(){
          System.out.println("函数已经完成返回");
      }
  }
  
  ```

  

* 注册bean到Spring容器，并添加aop配置

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xmlns:p="http://www.springframework.org/schema/p"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
              http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
              http://www.springframework.org/schema/context
              http://www.springframework.org/schema/context/spring-context-3.0.xsd
              http://www.springframework.org/schema/aop
              http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
              <context:property-placeholder location="classpath:information.properties"/>   
              <bean id="dahuang" class="com.yogurt.aop.YogurtServiceImpl">
                  <property name="name" value="${school}"></property>
              </bean>
              <bean id="myAdvice" class="com.yogurt.aop.MyAdvice"></bean>
              <aop:config>
                  <aop:aspect ref="myAdvice">
                      <aop:around method="around" pointcut="execution(* com.yogurt..*Impl.*(..))"/>
                      <aop:after-returning method="afterReturn" pointcut="execution(* com.yogurt..*Impl.*(..))"/>
                  </aop:aspect>
              </aop:config>
  </beans>
  ```

  啊

  ```properties
  #information.properties
  enName=propertiesYogurtBee
  school=propertiesSYSU
  ```

* 测试

  ```java
  public class TestAop {
      public static void main(String[] args){
         ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-aop.xml");
         YogurtService service = (YogurtService)ctx.getBean("dahuang");
         String ret = service.sayHello();
         System.out.println("return value = " + ret);
      }
  }
  ```

  结果

  ![1557637609215](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557637609215.png)



##### AOP常用标签说明

> * `<aop:config>`  作用：声明aop配置
>
> * `<aop:pointcut>` 作用：配置切入点表达式   属性：id：唯一标识切入点表达式名称 expression：定义切入点表达式
>
> * `<aop:aspect>` 作用：配置切面  属性：id：唯一标识切面的名称  ref：引用切面类（通知类）bean的id
> * `<aop:advisor>`作用：配置通知器（通知器和切面所实现的功能一样），属性：advice-ref 通知类bean
>
> * `<aop:before>` 作用：配置**前置通知**（在执行目标对象方法之前执行） 属性：method:指定通知方法名称 pointcut：定义切入点表达式  pointcut-ref:引用切入点表达式的id。
>
> * `<aop:after-returning>` 作用：配置**后置通知** 属性：method：指定通知方法名称 pointcut：定义切入点表达式 point-ref:引用切入点表达式的id
>
> * `<aop:after-throwing>` 作用：配置**异常通知**  属性：method：指定通知方法名称  pointcut:定义切入点表达式  pointcut-ref:引用切入点表达式的id
>
> * `<aop:after>` 作用：配置**最终通知** 属性：method：指定通知方法名称  pointcut：定义切入点表达式  pointcut-ref:引用切入点表达式的id
>
> * `<aop:around>` 作用：配置**环绕通知** 属性：method：指定通知方法名称  pointcut：定义切入点表达式  pointcut-ref:引用切入点表达式的id



##### `<aop:aspect>`与`<aop:advisor>`的区别

一般我们在面向切面编程是会使用`<aop:aspect>`，在进行事务管理时，一般会用`<aop:advisor>`

异同如下：

* aspect定义切面，通知器只需要是一般的bean就行了，而advisor标签引用的通知器，必须要实现Advice接口

  ```java
  //<aop:advisor>标签引用的通知类
  public class MyAdvice implements MethodBeforeAdvice{
      @Override
      public void before(Method m,Object[] arg1,Object arg2) throws Throwable{
          System.out.println("来自增强类");
      }
  }
  ```

  

  ```java
  //<aop:aspect>标签引用的通知类
  public class MyAdvice{
      public void before() throws Throwable{
          System.out.println("来自增强类");
      }
  }
  ```

* `<aop:advisor>`大多用于事务管理，`<aop:aspect>`大多用于日志和缓存

* 其实二者的基本原理是一样的，使用方式有所不同而已

  

##### 切入点表达式

* 语法

  > execution( 访问修饰符   方法返回值类型   包和类   方法 (方法参数)  方法声明的抛出异常 )

  其中，**方法返回值类型**，**方法**，是必填的

* 作用

  > 对符合切入点表达式的类，会自动生成代理对象

* 例子

  > * 最全的写法
  >
  >   execution(public void com.yogurt.aop.YogurtServiceImpl.sayHello())
  >
  > * 省略访问修饰符，返回值类型任意，say方法，有一个String参数
  >
  >   execution(* com.yogurt.aop.YogurtServiceImpl.say(java.lang.String))
  >
  > * 拦截com.yogurt包下，以及所有子包下的所有类的say方法
  >
  >   execution(* com.yogurt..*.say())
  >
  > * 拦截所有say方法 / 拦截所有方法
  >
  >   execution(* say())    //拦截所有say方法，这里省略了包和类
  >
  >   execution(* *())      //拦截所有方法
  >
  > * 拦截除say方法以外的所有方法
  >
  >   !execution(* say())
  >
  >    not execution(* say())   //注意not前面要有空格
  >
  > * 拦截say方法或者run方法
  >
  >   execution(* say()) || execution(* run())
  >
  >   execution(* say()) or execution(* run())
  >
  > * 拦截所有方法，参数有一个，参数类型任意
  >
  >   execution(* \*(*))
  >
  > * 拦截所有方法，参数任意，参数可有可无
  >
  >   execution(* *(..))
  >
  > * 最常用的写法
  >
  >   execution(* com.yogurt..*Impl.\*(..))      
  >
  >   //com.yogurt包下，及其所有子包下的以Impl结尾的类的所有方法
  >
  > 
  >
  > 方法参数表
  >
  > * \* ：代表一个任意类型的参数
  > * ..  :  代表0个或多个任意类型的参数
  >
  > 例如
  >
  > * ()  匹配无参方法
  > * (..)  匹配可接受任意数量和类型参数的方法
  > * (*)  匹配接收一个参数，参数类型任意的方法
  > * (*,int) 匹配接收两个参数，第一个参数类型任意，第二个是int的方法

#### AOP的使用（注解方式）

* 编写切面类（注意不是通知类）

  ```java
  package com.yogurt.aop.annotation;
  import org.aspectj.lang.ProceedingJoinPoint;
  import org.aspectj.lang.annotation.Around;
  import org.aspectj.lang.annotation.Aspect;
  import org.springframework.stereotype.Component;
  /** 
   * @ClassName: MyAspect 
   * @Description: TODO 
   * @author: yogurtbee 
   * @date: 2019-5-9
   * 
   */
  @Component
  @Aspect
  public class MyAspect {
      @Around(value = "execution(* com.yogurt.aop.YogurtServiceImpl.say*(..))")
      public Object dosomething(ProceedingJoinPoint joinPoint) throws Throwable{
          System.out.println("环绕注解aop...360度包围！");
          Object[] args = joinPoint.getArgs();
          Object returnVal = null;
          try {
              returnVal = joinPoint.proceed(args);
          } catch (Throwable e) {
              //
          }finally{
              
          }
          System.out.println("环绕注解aop...360度包围！");
          return returnVal;
      }
      
      //value可以是一个切入点表达式，也可以是一个切入点表达式的引用
      @Before(value = "MyAspect.pc()")
      public void before(){
          System.out.println("嘿嘿");
      }
      //也可以通过@Pointcut定义一个通用切入点
      @Pointcut(value = "execution(* *..*.say*(..))")
      public void pc(){}
  }
  
  ```

* 将切面类交给Spring容器管理

  ```xml
  <context:component-scan base-package="com.yogurt.aop.annotation"/>
  ```

* 开启aop自动代理

  ```xml
  <aop:aspectj-autoproxy/>
  ```

* 测试

```java
public static void main(String[] args){
       ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-aop-annotation.xml");
       YogurtService service = (YogurtService)ctx.getBean("dahuang");
       String ret = service.sayHello();
       System.out.println("return value = " + ret);
    }
```

结果

![1557637712636](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557637712636.png)



### 组件支撑

#### 整合Junit

- 问题

将测试类中的开头两行代码：

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-demo.xml");
MyService service = ctx.getBean("myService",MyService.class);
```

与业务代码解耦，让程序为我们自动创建spring容器

- 具体实现

  - 添加依赖

    > spring-test包
    >
    > junit包  （注意junit要4.0版本以上，才支持@RunWith注解）

  - 通过@RunWith注解，指定spring的运行器

    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = "classpath:spring-demo.xml")
    public class TestSpringJunit{
        @Autowired
        private MyService service;
        
        @Test
        public void testService(){
            service.execute();
        }
    }
    ```



#### 事务支持

事务最终都是交给数据库来实现的，但jdbc和Spring都可以对事务进行管理，管理的前提：事务是手动提交的。（自动提交的事务，咱们管不了）



##### 重要接口

* PlatformTransactionManager ：指定事务管理器要具备的功能
  * DataSourceTransactionManager：JDBC、Mybatis
  * HibernateTransactionManager：Hibernate
* TransactionDefinition：事务定义信息（隔离级别，传播行为，超时，只读等）
* TransactionStatus：事务的状态（是否是新事务，是否已提交，是否有保存点，是否回滚）
* **Spring是通过指定上述三个事务相关的接口，提供了对应的实现类，来对事务功能提供支持**

##### 事务管理方式

* 编程式事**务管理**（了解）

  Spring提供了模板类TransactionTemplate，故若要通过手动编程的方式，使用该模板即可，具体步骤如下：

  * 步骤一：配置一个事务管理器。Spring使用PlatformTransactionManager接口来管理事务，咱们要用到它的实现类，如DataSourceTransactionManager

  ```xml
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>
      </bean>
  ```

  * 步骤二：配置事务管理的模板

    ```xml
    <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
            <property name="transactionManager" ref="transactionManager"/>
        </bean>
    ```

  * 步骤三：在需要进行事务管理的类中，注入事务管理的模板

    ```xml
    <bean id="userService" class="com.yogurt.service.TestForTx">
            <property name="daoClient" ref="daoClient"/>
            <property name="transactionTemplate" ref="transactionTemplate"/>
        </bean>
        
        <bean id="daoClient" class="com.yogurt.dao.DaoClient"/>
    ```

  需要编写额外的非业务代码，去使用transactionManager去管理业务代码中的事务

  * 步骤四：在业务层去使用模板来管理事务

    ```java
    package com.yogurt.service;
    
    import com.yogurt.dao.DaoClient;
    import org.springframework.transaction.TransactionStatus;
    import org.springframework.transaction.support.TransactionCallback;
    import org.springframework.transaction.support.TransactionTemplate;
    
    public class TestForTx {
        private DaoClient daoClient;
        private TransactionTemplate transactionTemplate;
    
        //..省略了set/get方法
    
        public void testTx(){
            transactionTemplate.execute(new TransactionCallback<Object>() {
                @Override
                public Object doInTransaction(TransactionStatus transactionStatus) {
                    daoClient.insert("testTx","testTx123","I'm tx","tx@spring.io");
                    return null;
                }
            });
        }
    }
    
    //关键是借助于TransactionTemplate.execute(..)方法来执行事务管理
    //传入的参数可以又两种选择：
    //1. TransactionCallback           这个是有返回值的
    //2.TransactionCallbackWithoutResult    这个是无返回值的
    //可以调用TransactionTemplate的方法对事务传播级别，事务隔离级别进行设置
    ```

    

    关于上面代码出现的`new `后面跟`{}` 的情况的说明

    ```java
    public interface MyInt{
        void hello();
        void bye();
    }
    
    public class Test{
        @Test
        public void test(){
            MyInt impl = new MyInt(){
              @Override
                public void hello(){
                    System.out.println("hello");
                }
              @Override
                public void bye(){
                    System.out.println("bye");
                }
            };
            
           impl.hello();
        }
    }
    
    //这里的new后面跟大括号{}  ，实际是创建了一个匿名内部类，并且实例化了一个匿名内部类的对象
    //可以对 `接口` ， `抽象类`  使用这种方式来创建匿名内部类（实现类），并实例化一个对象
    
    //还有一种情况
    //也可以 new 一个具体的类(这个类既不是接口，也不是抽象类)，后面再跟大括号，如下,这里也相当于创建了一个匿名内部类（继承父类的子类），并实例化一个对象
    public class Yogurt{
        private String name;
        Yogurt(){}
        //..省略set/get方法
        public String haha(){
            return "haha";
        }
    }
    
    public class Test{
        @test
        public void test(){
            Yogurt newYogurt = new Yogurt(){
              @Override
                public String haha(){
                    return "fuck";
                }
            };
            String ret = newYogurt.haha();  //fuck
        }
    }
    ```

    

* 声明式事务管理（推荐）

* 