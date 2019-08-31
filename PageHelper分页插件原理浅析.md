PageHelper分页插件原理浅析

1. PageHelper分页插件的使用

   * 引入依赖

     ```xml
     <dependency>
     	<groupId>com.github.pagehelper</groupId>
     	<artifactId>pagehelper</artifactId>
     	<version>5.1.6</version>
     </dependency>
     ```

   * 在mybatis全局配置文件中添加plugin配置

     ```xml
     <configuration>
     	<properties resource="properties/db.properties"></properties>
         <!-- 注意plugins 标签的位置，需要放在environments标签之前 -->
     	<plugins>
     		<plugin interceptor="com.github.pagehelper.PageHelper">
     			<property name="dialect" value="mysql"/>
     		</plugin>
     	</plugins>
     	<!-- 数据源 -->
     	<environments default="development">
     		<environment id="development">
     			<transactionManager type="JDBC" />
     			<dataSource type="POOLED">
     				<property name="driver" value="${db.driver}" />
     				<property name="url" value="${db.url}" />
     				<property name="username" value="${db.username}" />
     				<property name="password" value="${db.password}" />
     			</dataSource>
     		</environment>
     	</environments>
     	<mappers>
     		<mapper resource="dao/UserMapper.xml" />
     	</mappers>
     </configuration>
     ```

   * 在DAO层进行分页查询

     ```java
     public class PageHelperTest{
         private SqlSessionFactory sqlSessionFactory;
         @Before
     	public void init() {
     		try {
     			String resource = "mybatis-config.xml";
     			InputStream inputStream = Resources.getResourceAsStream(resource);
     			sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
     		} catch (IOException e) {
     			e.printStackTrace();
     		}
     	}
         
         @Test
     	public PageInfo<User> testPage() {
     		SqlSession session = sqlSessionFactory.openSession();
             //开启分页，只对最近一次的查询有分页的效果
             //剧透一下，是使用了ThreadLocal来存储Page分页信息的
     		PageHelper.startPage(2,4);
             //下面的查询会被PageHelper拦截，返回的结果其实是一个Page对象，Page继承自ArrayList
     		List<User> lists = session.selectList("findAll");
             //mapper.xml里配置了一个<select id="findAll"> select * from user;</select>
     		PageInfo<User> pageInfo = new PageInfo<>(lists);
     		return pageInfo;
     	}
     }
     ```

2. PageHelper插件工作原理

   * 首先，在全局配置文件里使用`<plugin>`标签，将PageHelper拦截器添加到配置中。实际，拦截器是对Executor起了作用，先来看拦截器是如何被配置的：

     进行全局配置文件的解析和装载时，就会将拦截器信息封装到了Configuration对象中，流程如下：

     ```java
     String resource = "dao/SqlMapConfig.xml";
     InputStream inputStream = Resources.getResourceAsStream(resource);
     //下面的build方法，为全局配置文件的解析入口
     sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
     ```

     ```java
     public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
         try {
           XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
           // parser.parse()：使用XPATH解析XML配置文件，将配置文件封装为Configuration对象
           // 返回DefaultSqlSessionFactory对象，该对象拥有Configuration对象（封装配置文件信息）
           return build(parser.parse());
         } catch (Exception e) {
           throw ExceptionFactory.wrapException("Error building SqlSession.", e);
         } finally {
           ErrorContext.instance().reset();
           try {
             inputStream.close();
           } catch (IOException e) {
           }
         }
       }
     ```

     继续看XMLConfigBuilder中的parse方法

     ```java
     public Configuration parse() {
         if (parsed) {
           throw new BuilderException("Each XMLConfigBuilder can only be used once.");
         }
         parsed = true;
         parseConfiguration(parser.evalNode("/configuration"));
         return configuration;
     }
     
     private void parseConfiguration(XNode root) {
         try {
           //下面的解析顺序，即解释了配置文件中的标签书写顺序
           propertiesElement(root.evalNode("properties"));
           Properties settings = settingsAsProperties(root.evalNode("settings"));
           loadCustomVfs(settings);
           loadCustomLogImpl(settings);
           typeAliasesElement(root.evalNode("typeAliases"));
           // 关键在这，解析<plugins>标签
           pluginElement(root.evalNode("plugins"));
           objectFactoryElement(root.evalNode("objectFactory"));
           objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
           reflectorFactoryElement(root.evalNode("reflectorFactory"));
           settingsElement(settings);
           environmentsElement(root.evalNode("environments"));
           databaseIdProviderElement(root.evalNode("databaseIdProvider"));
           typeHandlerElement(root.evalNode("typeHandlers"));
           mapperElement(root.evalNode("mappers"));
         } catch (Exception e) {
           throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
         }
       }
     
     
     private void pluginElement(XNode parent) throws Exception {
         if (parent != null) {
           for (XNode child : parent.getChildren()) {
             //循环遍历plugins标签下的子标签
             String interceptor = child.getStringAttribute("interceptor");
             Properties properties = child.getChildrenAsProperties();
             Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
             interceptorInstance.setProperties(properties);
            //将当前拦截器添加到Configuration对象中
             configuration.addInterceptor(interceptorInstance);
           }
         }
       }
     
     //Configuration对象中
     public void addInterceptor(Interceptor interceptor) {
     	interceptorChain.addInterceptor(interceptor);
         //Configuration持有一个InterceptorChain对象
         //InterceptorChain中持有一个List<Interceptor>对象
     }
     ```

     

     至此，装载拦截器流程完毕，Configuration对象被封装到了DefaultSqlSessionFactory里

   * 使用拦截器对Executor进行增强

     之后，我们调用openSession打开一个SqlSession，看看创建SqlSession时都做了些什么

     ```java
     SqlSession session = sqlSessionFactory.openSession();
     ```

     ```java
     //DefaultSqlSession中
     public SqlSession openSession() {
         return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
     }
     
     private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
         Transaction tx = null;
         try {
           // 获取数据源环境信息
           final Environment environment = configuration.getEnvironment();
           final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
           tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
           // 创建Executor执行器，关键在这里*****
           final Executor executor = configuration.newExecutor(tx, execType);
           // 创建DefaultSqlSession
           return new DefaultSqlSession(configuration, executor, autoCommit);
         } catch (Exception e) {
           closeTransaction(tx); 
           throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
         } finally {
           ErrorContext.instance().reset();
         }
       }
     ```

     进入Configuration

     ```java
     public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
     		executorType = executorType == null ? defaultExecutorType : executorType;
     		executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
     		Executor executor;
     		if (ExecutorType.BATCH == executorType) {
     			executor = new BatchExecutor(this, transaction);
     		} else if (ExecutorType.REUSE == executorType) {
     			executor = new ReuseExecutor(this, transaction);
     		} else {
     			executor = new SimpleExecutor(this, transaction);
     		}
     		// 如果开启缓存（默认是开启的），则使用缓存执行器
     		if (cacheEnabled) {
     			executor = new CachingExecutor(executor);
     		}
     		// 插件增强，在这里使用插件对executor进行增强
     		executor = (Executor) interceptorChain.pluginAll(executor);
     		return executor;
     	}
     ```

     进入InterceptorChain

     ```java
     public Object pluginAll(Object target) {
         for (Interceptor interceptor : interceptors) {
           target = interceptor.plugin(target);
         }
         return target;
     }
     ```

     看到这里，就知道其实是通过Interceptor接口的plugin方法为纽带，来对Executor进行增强的（AOP)，这里，Interceptor即为PageHelper，所以接下来进入PageHelper

     ```java
     //PageHelper
     public Object plugin(Object target) {
        return target instanceof Executor ? Plugin.wrap(target, this) : target;
     }
     ```

     ```java
     //Plugin  
     //这个Plugin是mybatis自带的接口，实现了InvocationHandler，也说明mybatis拦截器底层是通过JDK动态代理来实现的
     public static Object wrap(Object target, Interceptor interceptor) {
         //前面入参过来，这里interceptor是PageHelper
         Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
         Class<?> type = target.getClass();
         Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
         if (interfaces.length > 0) {
             //下面是经典的JDK动态代理生成代码
             //继续进入Plugin的构造函数看看
           return Proxy.newProxyInstance(
               type.getClassLoader(),
               interfaces,
               new Plugin(target, interceptor, signatureMap));
         }
         return target;
     }
     
     private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
         this.target = target;
         this.interceptor = interceptor;
         this.signatureMap = signatureMap;
     }
     //只是进行了一些简单的变量设置
     
     //JDK动态代理，执行时会调用InvocationHandler中的invoke方法，所以我们接着看Plugin里面的invoke方法
     
     @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
         try {
           Set<Method> methods = signatureMap.get(method.getDeclaringClass());
           if (methods != null && methods.contains(method)) {
             //看到这里就比较明白了，实际调用了拦截器的intercept方法
             //那我们就去PageHelper的intercept方法里一探究竟
             return interceptor.intercept(new Invocation(target, method, args));
           }
           return method.invoke(target, args);
         } catch (Exception e) {
           throw ExceptionUtil.unwrapThrowable(e);
         }
     }
     ```

     啊

     ```java
     //PageHelper
     public Object intercept(Invocation invocation) throws Throwable {
             if (this.autoRuntimeDialect) {
                 SqlUtil sqlUtil = this.getSqlUtil(invocation);
                 return sqlUtil.processPage(invocation);
             } else {
                 if (this.autoDialect) {
                     this.initSqlUtil(invocation);
                 }
     
                 return this.sqlUtil.processPage(invocation);
             }
     }
     ```

     ```java
     //SqlUtil
     public Object processPage(Invocation invocation) throws Throwable {
             Object var3;
             try {
                 Object result = this._processPage(invocation);
                 var3 = result;
             } finally {
                 clearLocalPage();
             }
     
             return var3;
     }
     
     private Object _processPage(Invocation invocation) throws Throwable {
             Object[] args = invocation.getArgs();
             Page page = null;
             if (this.supportMethodsArguments) {
                 page = this.getPage(args);
             }
             RowBounds rowBounds = (RowBounds)args[2];
             if (this.supportMethodsArguments && page == null || !this.supportMethodsArguments && getLocalPage() == null && rowBounds == RowBounds.DEFAULT) {
                 return invocation.proceed();
             } else {
                 if (!this.supportMethodsArguments && page == null) {
                     //SqlUtil里维护了一个ThreadLocal<Page>
                     //这里会从ThreadLocal中去取Page对象
                     //这个Page对象是在调用PageHelper.startPage()时，被添加到ThreadLocal里的
                     page = this.getPage(args);
                 }
     
                 return this.doProcessPage(invocation, page, args);
             }
     }
     
     //下面是最最最终的执行
     //流程有点复杂
     //大概可以分为2步：
     //1. 修改入参中的MappedStatement，执行一次SQL查询，获取总数count,设置到Page对象中
     //（这一步可通过PageHelper.startPage(int pageNum,int pageSize,boolean count) 中的第三个参数来取消）
     //2. 通过Page对象里的分页信息，修改MappedStatement中的SqlSource，通过getBoundSql，在sql语句末尾添加 limit ?,? ，后通过StatementHandler，将分页信息绑定到入参，执行查询
     private Page doProcessPage(Invocation invocation, Page page, Object[] args) throws Throwable {
            //args是Executor执行query时的入参
            //args[0] 为 MappedStatement
            //args[1] 为 Object ，是SQL查询时的参数
            //args[2] 为 RowBounds 貌似和分页有关
            //agrs[3] 为ResultHandler 结果集处理器
            //可以参考CachingExecutor的query方法
             RowBounds rowBounds = (RowBounds)args[2];
             MappedStatement ms = (MappedStatement)args[0];
             if (!this.isPageSqlSource(ms)) {
                 this.processMappedStatement(ms);
             }
     
            //这里设置Parser，可能的取值有MysqlParser,OracleParser,PostgreSQLParser等
            //本例中为MysqlParser
             ((PageSqlSource)ms.getSqlSource()).setParser(this.parser);
     
             try {
                 args[2] = RowBounds.DEFAULT;
                 if (this.isQueryOnly(page)) {
                     Page var12 = this.doQueryOnly(page, invocation);
                     return var12;
                 }
     
                 if (page.isCount()) {
                     //如果开启了查询总数，先查询总数
                     page.setCountSignal(Boolean.TRUE);
                     //这里args[0]即是Executor执行查询时入参的MappedStatement
                     //先将MappedStatement入参改为查询总数的Statement
                     args[0] = msCountMap.get(ms.getId());
                     //下面执行Executor查询，得到总数
                     Object result = invocation.proceed();
                     //将Executor查询入参的MappedStatement恢复
                     args[0] = ms;
                     //将记录的总条数，设置到Page对象中
                     page.setTotal((long)(Integer)((List)result).get(0));
                     if (page.getTotal() == 0L) {
                         //若记录数量为0，则直接返回，不执行真正的查询
                         Page var7 = page;
                         return var7;
                     }
                 } else {
                     page.setTotal(-1L);
                 }
     
                 if (page.getPageSize() > 0 && (rowBounds == RowBounds.DEFAULT && page.getPageNum() > 0 || rowBounds != RowBounds.DEFAULT)) {
                     //存在分页信息时，进行分页查询
                     page.setCountSignal((Boolean)null);
                     //通过下面这一句，得到一个BoundSql，此时已经添加了limit ?,?
                     //args[1]是sql查询的参数，将会进行sql参数绑定
                     BoundSql boundSql = ms.getBoundSql(args[1]);
                     //这一句进行参数封装，将分页信息封装到args[1]中，后续会替换limit ?,?中的?
                     args[1] = this.parser.setPageParameter(ms, args[1], boundSql, page);
                     page.setCountSignal(Boolean.FALSE);
                     //执行查询
                     Object result = invocation.proceed();
                     //将结果封装为Page对象，并返回
                     page.addAll((List)result);
                 }
             } finally {
                 ((PageSqlSource)ms.getSqlSource()).removeParser();
             }
     
             return page;
     }
     ```

     

     下面通过一次debug过程，来从头到尾跟一遍PageHelper拦截器的执行

     下面是测试代码：

     ```java
     public class TestPageHelper{
     private SqlSessionFactory sqlSessionFactory;
     	@Before
     	public void before() {
     		try {
     			String resource = "dao/SqlMapConfig.xml";
     			InputStream inputStream = Resources.getResourceAsStream(resource);
     			sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
     		} catch (IOException e) {
     			e.printStackTrace();
     		}
     	}
     	@Test
     	public void testPage() {
     		SqlSession session = sqlSessionFactory.openSession();
     		PageHelper.startPage(2,4);
     		List<User> lists = session.selectList("findAll");
     		PageInfo<User> pageInfo = new PageInfo<User>(lists);
     		System.out.println(lists);
     	}
     }
     ```

     

     从SqlSessionFactory.openSession开始跟

     ![1566201858847.png](https://i.loli.net/2019/08/19/Gb5Mk2eYpfHPSFj.png)

     

     ![1566201943366.png](https://i.loli.net/2019/08/19/9pVUoWbItAKihfx.png)

     

     ![1566201992265.png](https://i.loli.net/2019/08/19/gKPowtSQmc8DU1X.png)

     

     ![1566202045498.png](https://i.loli.net/2019/08/19/ysiDMHaukOdmYEq.png)

     

     ![1566202160086.png](https://i.loli.net/2019/08/19/P9SkK2RDiTCVjdw.png)

     

     至此，openSession跟完了，使用JDK动态代理对Executor进行了增强

     

     下面开始跟PageHelper.startPage

     ![1566202298807.png](https://i.loli.net/2019/08/19/m5sFxIqajkw9JOQ.png)

     

     ![1566202432911.png](https://i.loli.net/2019/08/19/FfmaASMseh26OcN.png)

      

     

     下面开始跟SQL查询流程

     

     ![1566202501206.png](https://i.loli.net/2019/08/19/ifKprAMwLXlaUJ2.png)

     

     ![1566202670656.png](https://i.loli.net/2019/08/19/7x8X49TCtaByGfe.png)

     执行Executor的query之前，会执行PageHelper中的intercept方法，按照先前分析的，最终会进入到SqlUtil的doProcessPage方法，下面替换掉MappedStatement后，会执行Executor的query

     ![1566203370647.png](https://i.loli.net/2019/08/19/7JhbkPItfxDMugA.png)

     执行query

     ![1566202766551.png](https://i.loli.net/2019/08/19/vNm5GlroaXSp6dH.png)

     后委托给BaseExecutor执行查询

     ![1566202911212.png](https://i.loli.net/2019/08/19/7nvhkIrKXbJfoiB.png)

     最后会进入到BaseExecutor中的queryFromDatabase方法，随后进入SimpleExecutor的doQuery（这里不会进入缓存）

     ![1566203057268.png](https://i.loli.net/2019/08/19/ApM4iYHIgxVRErb.png)

     第一次查询总数完成后，继续返回到SqlUtil的doProcessPage方法中，继续往下执行

     ![1566204080033.png](https://i.loli.net/2019/08/19/3YpJVZMvidSugCs.png)

     先进入355，主要是将分页信息，封装到入参Object中

     ![1566204126557.png](https://i.loli.net/2019/08/19/VRPz1BpAq2OJQYj.png)

     

     ![1566204238628.png](https://i.loli.net/2019/08/19/V5qIdsaxRb9zYcp.png)

     

     ![1566204307524.png](https://i.loli.net/2019/08/19/GS4MrldOhi78pzu.png)

     下面进入到getBoundSql

     ![1566204350073.png](https://i.loli.net/2019/08/19/HvDJZojxCW6iSPX.png)

     继续进入PageRawSqlSource的getBoundSql

     ![1566204444377.png](https://i.loli.net/2019/08/19/fvrWLbJQ9KRslpY.png)

     继续看getPageBoundSql方法

     ![1566204502634.png](https://i.loli.net/2019/08/19/Y9XSaJ3sl21GAqn.png)

     继续看PageStaticSqlSource中的getPageBoundSql，关键在于调用了MysqlParser的getPageSql

     ![1566204585579](https://b2.bmp.ovh/imgs/2019/08/01c185224c0fc5f3.png)

     继续进入MysqlParser中去看看，最终找到了limit ? , ? 

     ![1566204678507](https://b2.bmp.ovh/imgs/2019/08/c48abb388a5bc2ef.png)

     再往后就是调用StatemenHandler对具体的SQL进行参数绑定，再执行查询，就不再赘述啦~

     下面列一下PageHelper涉及到的一些主要技术点作为总结：

     * 使用ThreadLocal存储Page分页信息（查询完后会清除）
     * 使用JDK动态代理来完成插件对Executor的拦截

     第一遍跟，会觉得头晕，多跟几遍，再梳理一下类的结构关系，就会比较清晰一点。一些更加细节的部分没有列出来，如PageHelper拦截后，将MappedStatement中的SqlSource再进行了一次封装，封装成PageSqlSource，PageRawSqlSource，还有PageStaticSqlSource啥啥的（在SqlUtil的`doProcessPage`中的`this.processMappedStatement()`方法中，这个方法内还有将`select count(0) from user`添加到一个map里的操作），以及第一次查询总数时，是如何将`select * from user`替换成`select count(0) from user`（在`SqlParser`中的`sqlToCount`方法），有兴趣的胖友可以自行深入。本人技术有限，如有纰漏或错误的地方，欢迎各位大佬批评指正。

     （完）