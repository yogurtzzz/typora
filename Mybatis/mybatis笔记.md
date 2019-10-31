[TOC]



# 主键返回

指的是将插入后生成的主键封装回传入的PO类

代码如下：

```java
class User{
    private String account;
    private int id;
    private String pass;
}
..............
    sqlSession = sqlSessionFactory.openSession();
	User user = new User();
	user.setAccount("test");
	user.setPass("test");
	UserMapper usermapper = sqlSession.getMapper(UserMapper.class);
	int rowsAffected = userMapper.insertUser(user);//方法调用的返回值是受影响行数
	sqlSession.commit();//提交事务
	System.out.println(user.getId());
```

其中UserMapper.xml配置如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 运用namespace来防止冲突 -->
<mapper namespace="com.yogurt.mapper.UserMapper">
    <insert id="insertUser" parameterType="com.yogurt.po.User" useGeneratedKeys="true" keyProperty="id">
		INSERT INTO userinfo (account,pass) VALUES (#{account},#{pass});
	</insert>
</mapper>
```

其中`insert`标签通过属性`useGeneratedKeys`和`keyProperty共同决定将自增的键封装回JavaBean的某个字段`

`useGeneratedKeys="true"` 使用生成的自增主键

`keyProperty="id"`将生成的自增主键封装回JavaBean的id字段



# 一对一关联查询

查询订单信息，以及每个订单对应的用户信息（一个订单对应一个用户）

## 使用扩展PO类

使用扩展PO类来接收SQL查询返回值，如用一个Order类来存储订单信息，新建一个OrderExt类（扩展Order，自动继承了Order的属性），并包含userAccount，userName信息。（OrderExt中的属性，必须和关联查询的返回结果字段值对应）

```xml
<!--一对一关联查询，使用简单扩展PO类来接收返回的结果集，该PO类需包含sql查询返回的所有字段信息-->
<!-- 注意下面resultType在全局配置文件中设置了typeAlias别名-->
	<select id="findOrdersByAccount" parameterType="java.lang.String" resultType="orderExt">
		SELECT  orders.*,userinfo.account userAccount,userinfo.nickName userName from orders,userinfo where orders.buyerId = userinfo.id AND userinfo.account = #{account}
	</select>
```

```java
public class Order {
    private int id;
    private String productName;
    private int price;
    private int buyerId;
    private int sellerId;
    private String buyerAddress;
	//省略了get/set函数
}
```



```java
public class OrderExt extends Order{
    //一对一关联查询中，可以定义专门的扩展po类作为输出结果类型
    //该po类中定义了sql查询结果集中所有的字段对应的属性
    //此方法比较简单，在企业中使用普遍
    private String userAccount;
    private String userName;
    //省略了get/set函数
}
```

数据库情况：

![1547979820735](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1547979820735.png)

DAO层代码：

```java
//注意是使用了mapper代理方式实现
public void findOrdersByAccount(String account){
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        try {
            List<OrderExt> res = userMapper.findOrdersByAccount(account);
            for (int i = 0; i < res.size(); i++){
                OrderExt item = res.get(i);
                System.out.println("Product : "+item.getProductName()+",Buyer : "+item.getUserName()+", Address : "+item.getBuyerAddress());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            sqlSession.close();
        }
    }
```

测试代码：

`daoTest.findOrdersByAccount("yogurt1");`

测试结果：

![1547979892292](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1547979892292.png)



## 使用resultMap

使用包含其他PO类对象的PO类来接受返回结果。在Mapper.xml映射文件中，需要使用resultMap和association标签

```java
public class OrderUserMapping {
    //一对一关联查询
    //使用resultMap和association
    private Order order;
    private User user;
	//省略了set/get方法
}
```

mapper.xml配置文件：

```xml
<!--一对一关联查询，使用嵌套，即PO类中包含了其他PO类对象，使用resultMap和association来进行映射 -->
	<resultMap id="one2oneMap" type="orderUserMapping">
		<association property="user" javaType="user">
			<id column="buyerId" property="id"></id>
			<result column="buyerAccount" property="account"></result>
			<result column="buyerName" property="nickName"></result>
		</association>
		<association property="order" javaType="order">
			<id column="id" property="id"></id>
			<result column="productName" property="productName"></result>
			<result column="price" property="price"></result>
			<result column="buyerId" property="buyerId"></result>
			<result column="sellerId" property="sellerId"></result>
			<result column="buyerAddress" property="buyerAddress"></result>
		</association>
	</resultMap>
	<select id="findOrdersWithMap" parameterType="java.lang.String" resultMap="one2oneMap">
		SELECT orders.*,userinfo.account buyerAccount,userinfo.nickName buyerName FROM orders LEFT JOIN `userinfo`  ON orders.buyerid=userinfo.id where userinfo.account=#{account}
	</select>
```

DAO层代码：

```java
public void findOrdersWithMap(String account){
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        try {
            List<OrderUserMapping> res = userMapper.findOrdersWithMap(account);
            for (int i = 0;i < res.size(); i++){
                OrderUserMapping item = res.get(i);
 	System.out.println(item.getOrder().getProductName()+","+item.getUser().getNickName());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            sqlSession.close();
        }
    }
```

测试代码：

`daoTest.findOrdersWithMap("yellow123yellow");`

测试结果：

![1547980467395](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1547980467395.png)



# 一对多查询

查询用户信息，以及每个用户下对应的订单信息（一个用户对应多个订单）

1. **只能使用resultMap进行查询**
2. **sql返回语句可能有多个，但是映射的对象是一个**
3. 

# 批量查询

* 动态SQL（where/if/sql片段/include/foreach）
* 注意事项：foreach标签使用的时候，有个collection属性，该属性值的写法要注意
  * 若parameterType为java.util.List或java.util.Array的话，那么属性值必须是list或者array，除此之外
  * 若parameterType为POJO类型，那么属性值必须是该POJO类中的一个List类型或Array数组变量名称
* 多表关联查询
  * resultMap标签，以及id和result字标签
  * 一对一关联，在resultMap标签中，使用association子标签表示一对一关系
  * 一对多关联，在resultMap标签中，使用collection子标签表示一对一关系
  * 注意事项：结果集java对象的嵌套关系，直接表现为resultMap和association或者collection子标签的嵌套关系
* 查询缓存
  * 在mapper映射文件中配置`<cache>`标签
  * 二级缓存作用于：namespace级别
* 延迟加载（预先加载主信息，延迟加载从信息）
  * 侵入式延迟加载：加载主信息，如果get主信息中的字段，它会将从信息进行加载
  * 深度延迟加载：加载主信息，如果get主信息中的字段，它不会将从信息进行加载，只有get从信息中的字段时，才会进行加载从信息
* XML开发方式和Annotation开发方式
* 逆向工程（mybatis提供的）
  * 针对单表

# 源码分析

## 阅读方法

1. 找主线
2. 找入口
3. 记笔记（类名#方法名）
4. 参考其他人阅读源码的经验

## 阅读目的

* 提升对设计模式的理解，提升编程能力
* 通过阅读源码，找到问题根源，以便解决问题
* **应付面试**

## 源码阅读

### 加载全局配置文件的流程

* 找入口：SqlSessionFactory#build方法

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
        SqlSessionFactory var5;
        try {
            //XMLConfigBuilder： 用来解析XML配置文件
            XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
            //paser.parse() : 使用Xpath解析XML配置文件，将配置文件封装为Configuration对象
            //build(Configuration)方法，返回一个DefaultSqlSessionFactory
            var5 = this.build(parser.parse());
        } catch (Exception var14) {
            throw ExceptionFactory.wrapException("Error building SqlSession.", var14);
        } finally {
            ErrorContext.instance().reset();

            try {
                inputStream.close();
            } catch (IOException var13) {
                ;
            }

        }

        return var5;
    }
public SqlSessionFactory build(Configuration config) {
        return new DefaultSqlSessionFactory(config);
    }
```

* 调用关系整理

  ```reStructuredText
  SqlSessionFactory#build()：用于构建SqlSessionFactory对象
  	|-- XMLConfigBuilder#构造函数: 用来解析全局配置文件的解析器
  		|-- XPathParser#构造函数：使用XPath语法来解析XML的解析器
  			|-- XPathParser#createDocument：解析全局配置文件，封装为Document对象
  		|-- Configuration#构造函数：用来创建Configuration对象，同时初始化内置类的别名 
  	|-- XMLConfigBuilder#parse():全局配置文件的解析器
  		|-- XPathParser#evalNode("/configuration"):  XPath解析器，通过XPath语法来解析XML，参数是一个XPath语法的字符串，返回值是XNode对象
  		|-- XMLConfigBuilder#parseConfiguration(XNode)：从全局配置文件根节点开始解析,并将解析出的数据封装到Configuration对象中 (其中对**mappers标签**的解析，放在下一节单独说)
  	
  	|-- SqlSessionFactory#build(Configuration)：创建SqlSessionFactory接口的默认实现类DefaultSqlSessionFactory（Configuration对象）
  
  
  SqlSessionFactoryBuilder创建SqlSesssionFactory时，需要传入一个Configuration对象。
  XMLConfigBuilder会去创建Configuration，并且通过XPathParser去解析全局配置文件，形成Document对象和XNode节点
  XMLConfigBuilder对象会针对Document对象中的指定XNode节点进行解析，解析后的数据，设置到Configuration对象中 
  
  -> 通过UML建模语言，绘制源码的时序图
  ```

  

### 加载映射文件的流程

* 入口 XMLConfigBuilder#mapperElement方法

  ```java
  private void mapperElement(XNode parent) throws Exception {
          if (parent != null) {
              Iterator var2 = parent.getChildren().iterator();
  
              while(true) {
                  while(var2.hasNext()) {
                      XNode child = (XNode)var2.next();
                      String resource;
                      if ("package".equals(child.getName())) {
                          resource = child.getStringAttribute("name");
                          this.configuration.addMappers(resource);
                      } else {
                          resource = child.getStringAttribute("resource");
                          String url = child.getStringAttribute("url");
                          String mapperClass = child.getStringAttribute("class");
                          XMLMapperBuilder mapperParser;
                          InputStream inputStream;
                          if (resource != null && url == null && mapperClass == null) {
                              ErrorContext.instance().resource(resource);
                              inputStream = Resources.getResourceAsStream(resource);
                              mapperParser = new XMLMapperBuilder(inputStream, this.configuration, resource, this.configuration.getSqlFragments());
                              mapperParser.parse();
                          } else if (resource == null && url != null && mapperClass == null) {
                              ErrorContext.instance().resource(url);
                              inputStream = Resources.getUrlAsStream(url);
                              mapperParser = new XMLMapperBuilder(inputStream, this.configuration, url, this.configuration.getSqlFragments());
                              mapperParser.parse();
                          } else {
                              if (resource != null || url != null || mapperClass == null) {
                                  throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
                              }
  
                              Class<?> mapperInterface = Resources.classForName(mapperClass);
                              this.configuration.addMapper(mapperInterface);
                          }
                      }
                  }
  
                  return;
              }
          }
      }
  ```

* 调用关系整理

  ```reStructuredText
  XMLConfigBuilder#parseConfiguration("\configuration"根节点)中
  	|-- XMLConfigBuilder#mapperElement("\mappers"节点) ： 解析全局配置文件中的<mappers>子标签
  		|-- XMLMapperBuilder#构造函数：专门用来解析映射文件
  			|-- XPathParser#构造函数：
  				|-- XPathParser#createDocument : 创建mapper映射文件对应的document对象
  			|-- MapperBuilderAssistant#构造函数 ： 用于构建MappedStatement对象
  		|-- XMLMapperBuilder#parse：
  			|-- XMLMapperBuilder#configurationElement：专门用来解析mapper映射文件
  				|-- XMLMapperBuilder#buildStatementFromContext ： 用来创建MappedStatement对象
  					|--XMLMapperBuilder#buildStatementFromContext
  						|-- XMLStatementBuilder#构造方法 ：专门用来解析MappedStatement
  						|-- XMLStatementBuilder#parseStatementNode：
  							|-- MapperBuilderAssistant#addMappedStatement：创建MappedStatement对象
  								|-- MappedStatement.Builder#构造方法
  								|-- MappedStatement.Builder#build方法：创建MappedStatement对象，并将创建的MappedStatement对象放在Configuration对象中
  ```

  全局配置文件中的mappers标签用法总结：

  使用<package>子标签加载Mapper接口 : mapper映射文件和mapper接口必须同包同名

  mapper接口和mapper映射文件还必须满足4个规范：

  1. mapper接口全路径名称  与 mapper映射文件的namespace相同
  2. mapper方法名   与    mapper映射文件的CRUD标签的id名相同
  3. mapper参数类型     .... 
  4. mapper返回值类型   .....



### SqlSession执行主流程

### 参数绑定流程

### 结果集映射流程



# 设计模式篇

## 构建者模式

## 工厂模式

## 代理模式

## 委托模式

# 手写框架篇