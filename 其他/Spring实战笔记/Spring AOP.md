# Spring AOP

引言：

一个系统中，有的功能会被应用到各个模块当中，但是我们又不想在每个点明确的调用它们。比如**日志，安全，事务管理**，它们都很重要，但不应该由各个模块来主动参与它们的行为，各个模块应该只关注自己的业务逻辑，这部分公共的问题，应该由其他对象来单独处理...

**横切关注点**：指的是散布于系统各个地方的通用功能，比如**日志，安全，事务管理**。这些通用功能，从概念上讲，应该是和业务逻辑相分离的。而**AOP**就是要解决这个问题。

Spring的DI，有助于**系统中各个bean对象的解耦**

而AOP，有助于将**横切关注点与业务逻辑解耦**



要进行功能的重用，最常见的技术是**继承**，但是如果在所有类中使用相同的基类，会造成一个脆弱的对象体系，并且继承表达的是一种`is a`的关系，但在这种场景下，并不是`is a `的关系



## AOP术语:

- JoinPoint 连接点

  系统中可以应用通知的时机，被称为连接点。也就是可以被AOP拦截的地方

  连接点可以分类为：

  - **方法连接点**
  - 字段连接点
  - 构造器连接点

  Spring只支持方法级别的连接点，AspectJ可支持下面2种

- PointCut 切入点

  可以理解为连接点的**匹配条件**。满足切入点条件的一个或多个连接点，将会被应用通知。即，指定了哪些连接点，将会被AOP拦截。

- Advice 通知

  描述了切面所要完成的工作，以及在何时执行这些工作。

  包含2个点：

  - 切面**是什么**
  - **何时**起作用
    - 前置通知（Before）
    - 后置通知（After）
    - 环绕通知（Around）
    - 返回通知（After-Returning）
    - 异常通知（After-Throwing）

- Aspect 切面

  切面 = 切入点 + 通知

  - **切入点**定义了切面在**何处**被使用
  - **通知**定义了切面需要**做什么**，以及在**何时**做

  总结起来就是，在**何处**，**何时**，**做什么**

- Introduction 引介

  引介允许我们向现有的类添加新方法或属性，可以在无需修改现有类的情况下，让它们具有新的行为和状态

- Weaving 织入

  将**切面**应用到**目标对象**，并创建新的代理对象的**过程**。

  切面在指定的连接点，被织入到目标对象中。在目标对象的生命周期中有多个时间点可以进行织入：

  - 编译期

    在目标类编译时织入，需要特殊的编译器，AspectJ的织入即是这种方式

  - 类加载期

    在目标类被加载到JVM时织入，需要特殊的类加载器，在目标类被加载时，增强该目标类的字节码

  - 运行期

    切面在运行时被织入，AOP容器将动态创建一个代理对象。Spring AOP即是这种方式



## Spring对AOP的支持

- **基于代理的经典Spring AOP**

  使用ProxyFactoryBean，现已很少使用

- 纯POJO的切面

  切面就是个普通java类，通过在XML中使用aop命名空间来将POJO转换成切面

- @Aspect注解驱动的切面

  使用AspectJ注解驱动风格来实现切面，但依然是基于代理的Spring AOP

- 注入式AspectJ切面

  真正的AspectJ

前3种都是属于Spring AOP的实现，Spring AOP构建于 动态代理基础上，所以其仅限于**对方法的拦截**

## AOP使用

- 切入点表达式  PointCut Expression

  - designators 指示器

    - `execution()`

      格式：  `execution([限定符(可选)][返回值类型] [指定包下的类] [方法] [形参] [异常声明(可选)])`

      如： `execution(public * com.service..*.*(..))` 

      表示匹配任意返回值类型，com.service包以及其子包下的所有类的所有方法，形参任意

    - `within()`：匹配包，或某一具体类型（若指定为接口并不能拦截到接口的实现类）

      ```java
      	//匹配com.service包下的所有类中的方法
      	@Pointcut("within(com.service.*)")
          public void service(){}
      	
      	//匹配com.service包下所有以Service结尾的类的方法
      	@Pointcut("within(com.service.*Service)")
          public void suffixService(){}
      
          @Before("service()")
          public void aop(){
              System.out.println("aop take effect");
          }
      
      //注意within用来匹配接口是无效的
      //假设我们期望匹配com.service下所有实现了SomeService接口的类
      //如下的写法并不能达成目标，将within换成this，target，或者使用更通用的execution即可
      //或者在within里面的接口后面加+ ，表示匹配类及其子类
      	@Pointcut("within(com.service.SomeService)")
          public void service(){}
      ```

      

    - `this()`：匹配目标对象的AOP代理对象，可以拦截到introduction中的方法

      ```java
      	//拦截SomeService接口的所有实现类（就算实现类不在com.service包下）
      	@Pointcut("this(com.service.SomeService)")
          public void service(){}
      
          @Before("service()")
          public void aop(){
              System.out.println("aop take effect");
          }
      ```

      

    - `target()`：匹配目标对象，不能拦截introduction中的方法，在没有引入introductio时，和this是一样的

    - `bean()`：匹配spring IoC容器里的bean(通过bean名称)

      ```java
      	//匹配bean名称为dinnerService的bean对象中只有一个参数且为Long的方法
      	@Pointcut("bean(dinnerService) && args(Long)")
          public void service(){}
      ```

      

    - `args()`：匹配参数

      ```java
      	//匹配SomeService接口的所有实现类中，只有一个参数，且为Long的方法
      	@Pointcut("this(service.SomeService) && args(Long)")
          public void service(){}
      	//匹配SomeService接口的所有实现类中，只有一个参数且为Long，或者第一个参数为Long的方法
      	@Pointcut("this(service.SomeService) && args(Long,..)")
          public void service2(){}
      
          @Before("service()")
          public void aop(){
              System.out.println("aop take effect");
          }
      ```

    - `@annotation()`：匹配带有某一注解的方法

      ```java
      @Aspect
      @Component
      public class YogurtAOP {
          @Pointcut("@annotation(anno.Yogurt)")
          public void service(){}
      
          @Before("service()")
          public void aop(JoinPoint joinPoint) throws Throwable {
              Method m = joinPoint.getTarget().getClass().getMethod(joinPoint.getSignature().getName());
              if (m.isAnnotationPresent(Yogurt.class)){
                  Yogurt yogurt = m.getAnnotation(Yogurt.class);
                  String val = yogurt.value();
                  System.out.println(val);
              }
              System.out.println("aop take effect");
          }
      }
      ```

      

      ```java
      @Component
      public class DinnerService implements SomeService {
          @Override
          @Yogurt("fantastic")
          public void doService() {
              System.out.println("Serve dinner");
          }
      }
      ```

      

      结果：

      ```
      fantastic
      aop take effect
      Serve dinner
      ```

      

    - `@within()` 匹配带有某一注解的类

      ```java
      	@Pointcut("@within(anno.CheckThis)")
          public void service(){}
      ```

      

      ```java
      @Component
      @CheckThis
      public class DinnerService implements SomeService {
          @Override
          public void doService() {
              System.out.println("Serve dinner");
          }
      }
      ```

      

    - `@target()` 匹配带有某一注解的类 

    

    

    

    - `@args()` 匹配带有某一注解的参数的方法

      注意，不是在方法的形参列表前带注解，而是方法的入参类型，类型定义上带有注解

      ```java
      //如，有一个这样的方法
      public void service(@CheckThis Args args){
          
      }
      
      @Pointcut("@args(com.anno.CheckThis)")
      //并不能拦截到service方法
      //正确的使用如下
      
      @CheckThis
      public class Args{
          
      }
      
      public void service(Args args){
          
      }
      
      @Pointcut("@args(com.anno.CheckThis)")
      
      
      ```

      

  - wildcard 通配符

    - `*`   匹配任意数量字符
    - `+`   匹配指定类及子类
    - `..`  匹配包及子包，或任意参数

  - operators 运算符

    - `&&`  ：在XML配置中，使用`and`
    - `||` ：在XML配置中，使用`or`
    - `!`  ：在XML配置中，使用 `not`

  

- 参数传递

  ```java
  	//匹配com.service.SomeService接口的全部实现类，中的只有一个Long参数的方法，并绑定其参数
  	@Pointcut("target(com.service.SomeService) && args(number)")
      public void service(Long number){}
  
      @Before("service(number)")
      public void aop(Long number){
          System.out.println("aop take effect with number " + number);
      }
  ```

- 引介

  Spring AOP实际是生成了一个代理对象，这个代理对象包装了目标对象，若要为这个目标对象添加新的方法或字段，可以通过**引介**来实现。其实质是将新的方法或属性，用一个新的bean去实现，然后代理对象同时包装了目标对象和这个新的bean。即一个bean的实现被拆分到多个类中。调用代理对象的方法时，会先判断方法是原始目标对象的，还是通过引介引入的新方法...代理对象再委托给其内部的不同bean去执行

  使用方法：

  - 创建一个切面

    ```java
    @Aspect
    @Component
    public class YogurtIntroduction {
        @DeclareParents(value ="service.DinnerService+",defaultImpl = NewFunctionImpl.class)
        public static NewFunction function;
        @Before("execution(* service.DinnerService.*(..)) && this(function)")
        public void before(NewFunction function){
            function.fun();
        }
    }
    ```

  - 创建一个新的接口和其实现类

    ```java
    public interface NewFunction {
        public void fun();
    }
    
    public class NewFunctionImpl implements NewFunction {
        @Override
        public void fun() {
            System.out.println("this is method by introduction");
        }
    }
    ```

  - 测试

    ```java
        @Autowired
        ApplicationContext context;
    
        @Test
        public void test(){
            SomeService dinnerService = (SomeService) context.getBean("dinnerService");
            assertNotNull(dinnerService);
            dinnerService.doService(1L);
    
            NewFunction function = (NewFunction) dinnerService;
            function.fun();
        }
    ```

    

## Spring AOP 与 AspectJ

- 目标不同
  - Spring AOP旨在提供一个简单的AOP实现，以解决一些常见的问题，但这并不是完整的AOP解决方案，但**简单**方便，开发人员上手快，Spring AOP只支持方法级别的连接点。且只能在Spring框架中，借助IoC容器使用
  - AspectJ是最成熟的AOP框架，提供了更完整的AOP解决方案，但是也更加**复杂**，需要引入许多AOP工具集，如AJC编译器，有一定学习成本。AspectJ支持所有级别的连接点，如字段连接点，构造器连接点等
- 织入时机
  - Spring AOP是动态织入
  - AspectJ是静态织入，可以在编译期织入，或类加载时织入
- 性能
  - AspectJ由于是静态织入，性能会更快
  - Spring AOP是动态织入，基于代理，运行时会涉及到代理对象的生成，切面中还有一些方法调用，会对性能造成一定影响。据评测，AspectJ大概会被Spring AOP快8-35倍

| Spring AOP                             | AspectJ                                                      |
| -------------------------------------- | ------------------------------------------------------------ |
| 用标准JAVA实现                         | 用JAVA语言扩展实现                                           |
| 不需要特殊的编译器                     | 需要引入AJC编译器                                            |
| 仅支持运行时织入                       | 仅支持编译时织入，或类加载时织入                             |
| 仅支持方法级别织入，less powerful      | 支持字段，方法，构造器，静态字段，final 修饰的类/方法的织入，more powerful |
| 需要借助Spring IoC容器中的bean实现     | 可以被任意的bean实现                                         |
| 生成目标对象的代理，并将切面应用于代理 | 切面在应用运行之前，被直接织入到字节码中                     |
| 性能更慢                               | 性能更快                                                     |
| 学习成本低，简单易上手                 | 学习成本高，较复杂                                           |







