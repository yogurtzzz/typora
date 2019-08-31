mybatis加载配置文件源码

* 入口：SqlSessionFactoryBuilder.build()，入参是一个InputStream

  ```java
  private SqlSessionFactory sqlSessionFactory;
      @Before
      public void init()throws Exception {
          String resource = "mybatis-config.xml";
          InputStream inputStream = Resources.getResourceAsStream(resource);
          /** 下面是入口 **/
          sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
      }
  ```

* SqlSessionFactoryBuilder.build内部

  * 创建一个XMLConfigBuilder对象

    * 先创建一个XPathParser（这一步会创建Document对象），并使XMLConfigBuilder持有这个XPathParser对象。这个XPathParser会在之后用来解析XML文档
    * 再调用XMLConfigBuilder的构造方法（Configuration对象在这一步被创建）
    * 注意：XMLConfigBuilder继承自BaseBuilder，在BaseBuilder里有Configuration，TypeAliasRegistry，TypeHandlerRegistry

  * 调用XMLConfigBuilder对象上的parse方法

    * 调用`this.parseConfiguration(parser.evalNode("/configuration"))`，从根标签`<configuration>`开始解析

      * 该方法内部即是对全局配置文件中各个标签进行解析

        

    * 上面一步会将解析的结果封装到Configuration对象中，解析完成后返回Configuration对象

  * parse方法返回一个Configuration对象，以此Configuration对象为参数，创建一个DefaultSqlSessionFactory，并返回