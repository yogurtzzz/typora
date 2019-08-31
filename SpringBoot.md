# SpringBoot

第一次搭建SpringBoot项目时，报错

`java.lang.NoClassDefFoundError: ch/qos/logback/classic/Level`

原因：jar包不完整

-> 解决方法：将maven库里对应的jar包删除，在IDEA中重新maven update一下，重新下载jar包



SpringBoot的启动方式

* IDEA中直接运行xxxxApplication类
* 通过maven启动：在命令行下进入到工程目录，输入命令`mvn spring-boot:run`
* 通过maven打包后再启动：进入到工程目录，输入命令`mvn clean package`[^1]，打包完成后，通过`java -jar xxxx.jar`启动



## 项目属性配置

YAML文件里的自定义属性注入：

```yaml
## yml文件
server:
  port: 8023
  servlet:
    context-path: /luckey

limit:
  min: 10
  max: 99999
  description: 最少要发${limit.min}元，最多能发${limit.max}元
```



* 通过`@Value`注入

  ```java
  @RestController
  public class HelloController{
      @Value("${limit.min}")
      private BigDecimal min;
      @Value("${limit.max}")
      private BigDecimal max;
  }
  ```

  

* 通过对象注入：

  ```java
  @Component
  @ConfigurationProperties(prefix = "limit")
  @Getter
  @Setter
  public class LimitConfig {
      /**属性的字段名要和yml文件里的属性名对应**/
      private BigDecimal min;
      private BigDecimal max;
      private String description;
  
  }
  
  @RestController
  public class HelloController{
      @Autowired
      private LimitConfig config;
      
      @GetMapping("/yogurt")
      public String hello(){
          return config.getDescription();
      }
  }
  ```

  

多环境的配置：

配置开发和生成环境的yml文件：

新建2个yml文件并完成配置：

* application-dev.yml
* application-prod.yml

在application.xml中配置：

```yaml
spring:
  profiles:
    active: dev
# 则默认使用开发环境的配置，即使用application-dev.yml
```

然而在**开发环境**与**生产环境**之间切换时，不需要每次修改application.yml，仍然按上面的设置，在启动时，传入参数即可，使用下面的命令启动将替换为生产的配置，这里的D是properties的含义  [参考链接](https://blog.csdn.net/yy193728/article/details/72847122)

`java -jar -Dspring.profiles.active=prod luckeymoney-0.0.1-SNAPSHOT.jar`



## Controller的使用

* `@RestController`：作用在类上，等同于`@Controller` + `@ResponseBody`

* `@PathVariable`：获取url中的数据，比如url为`xxxxx/pages/3`，这个具体第几页的参数，就可以用该方法传进去

  ```java
  @RestController
  public class HelloController {
  
      @GetMapping("/yogurt/{data}")
      public String hello(@PathVariable("data") String data){
          return "入参信息"+data;
      }
  }
  
  //打开浏览器，输入地址  localhost:8080/yogurt/hello
  //即会打印出     入参信息hello
  ```

  

* `@RequestParam`：获取请求参数中的数据

  ```java
  @RestController
  public class HelloController {
  
      @GetMapping("/yogurt")
      public String hello(@RequestParam("data") String data){
          return "入参信息"+data;
      }
  }
  
  //打开浏览器，输入地址  localhost:8080/yogurt?data=hello
  //即会打印出     入参信息hello
  
  //若是非必传参数，可以设置@RequestParam的required属性为false，以及可以设置默认属性
  @GetMapping("/yogurt")
      public String hello(@RequestParam(value = "data", required = false, defaultValue = "default") String data){
          return "入参信息"+data;
      }
  ```



## JPA

### Optional

java 8 新特性，可以让代码更简洁，[参考链接](https://blog.csdn.net/zknxx/article/details/78586799)



[^1]:`mvn clean package` / `mvn clean install` / `mvn clean deploy`的区别，`package`仅仅完成编译打包，并不会将打好的包发布到本地maven库；`install`编译打包完成后，会将包发布到本地maven库；`deploy`编译打包完成后，会将包发布到本地maven库，同时会部署到远程maven库。详情参考maven的生命周期[参考链接](https://blog.csdn.net/zhaojianting/article/details/80324533)