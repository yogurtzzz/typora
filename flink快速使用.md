快速使用 flink

1. 使用 maven 骨架构建一个 flink项目

   ![image-20200706171735936](C:\Users\iGola\AppData\Roaming\Typora\typora-user-images\image-20200706171735936.png)

2. 配置 flink 任务并启动

   ```java
   package com.yogurt.flink;
   
   import org.apache.flink.api.common.functions.FlatMapFunction;
   import org.apache.flink.api.java.DataSet;
   import org.apache.flink.api.java.ExecutionEnvironment;
   import org.apache.flink.api.java.tuple.Tuple2;
   import org.apache.flink.util.Collector;
   public class BatchJob {
   
   	public static void main(String[] args) throws Exception {
   		// set up the batch execution environment
   		final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
           DataSet<String> stringDataSet = env.fromElements("this,is",
                   "yogurt,saying,hello");
   
           DataSet<Tuple2<String,Integer>> wordCount = stringDataSet
                   .flatMap(new FlatMapFunction<String, Tuple2<String,Integer>>() {
                       @Override
                       public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                           for (String is : s.split(",")) {
                               collector.collect(new Tuple2<>(is,1));
                           }
                       }
                   })
                   .name("FLAT MAP")
                   .groupBy(0)
                   .sum(1);
   
           wordCount.print();
           //如果不写下面这一句, 会报错说没有 data sink
           wordCount.writeAsText("E:\\output");
   		env.execute("Flink Batch Java API Skeleton");
   	}
   }
   //可以直接在idea中运行, 也可以打成jar包后,通过flink-web上传到 flink服务端运行
   
   ```

3. 从mysql加载数据源

   ```java
   final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
   //核心是调用env的createInput
   env.createInput();
   //给createInput方法传入一个InputFormat参数
   //可以用如下方式创建一个 JDBCInputFormat
   //其中需要指定 url , user , password , 查询用的sql语句, 以及返回参数的TypeInfo
   JDBCInputFormat jdbcInputFormat = JDBCInputFormat.buildJDBCInputFormat()
                   .setDrivername("com.mysql.jdbc.Driver")
                   .setDBUrl(url)
                   .setUsername(user)
                   .setPassword(pass)
                   .setQuery(query)
                   .setRowTypeInfo(rowTypeInfo)
                   .finish();
   //如果查询语句如下：
   //select id,lat,lgt from data_service.flights_airport where status ='ENABLED';
   //那么rowTypeInfo应该如下, (查询结果是3列)
   TypeInformation[] fieldTypes =new TypeInformation[]{
                   BasicTypeInfo.STRING_TYPE_INFO,
                   BasicTypeInfo.STRING_TYPE_INFO,
                   BasicTypeInfo.STRING_TYPE_INFO
           };
   RowTypeInfo rowTypeInfo =new RowTypeInfo(fieldTypes);
   
   //完整的使用如下：
   //通过env.createInput创建一个输入源
   //设置并行度为1
   //调用map转换算子, 将查询结果转换为POJO
   DataSet<AirportData> airportDataSet = env.createInput(QueryUtils.executeMysqlQuery(mysqlHost, mysqlUser, mysqlPass, sql))
   					.setParallelism(1)
   					.map(new AirportDataMapFunction())
   					.name("GETTING AIRPORT LAT & LGT");
   
   //其中的MapFunction如下
   public class AirportDataMapFunction implements MapFunction<Row, AirportData> {
   
       private static final Logger logger = LoggerFactory.getLogger(AirportDataMapFunction.class);
       @Override
       public AirportData map(Row row) throws Exception {
           if (StringUtils.isEmpty((String)row.getField(0))) {
               return AirportData.builder()
                       .id("default")
                       .lat(.0f)
                       .lgt(.0f)
                       .build();
           }
           return AirportData.builder()
                   .id(String.valueOf(row.getField(0)))
                   .lat(Float.parseFloat((String)row.getField(1)))
                   .lgt(Float.parseFloat((String)row.getField(2)))
                   .build();
       }
   }
   ```

4. 从Cassandra读取数据源

   ```java
   DataSet<OAGData> oagDataSet = env.createInput(QueryUtils.executeQuery(OAGCql, oagHost, OAGData.class),TypeInformation.of(new TypeHint<OAGData>() {})).name("GETTING OAG DATA for date:" + avDepDate);
   
   public static CassandraInputFormatBase executeQuery(String cql, String clusters, Class clazz){
           ClusterBuilder builder = new ClusterBuilder() {
               private static final long serialVersionUID = 1;
               @Override
               protected Cluster buildCluster(com.datastax.driver.core.Cluster.Builder builder) {
                   return builder.addContactPoints(clusters)
                           .build();
               }
           };
           if(clazz != null) {
               return new CassandraPojoInputFormat(cql, builder, clazz);
           }else{
               return new CassandraInputFormat<>(cql, builder);
           }
       }
   ```

5. 使用Flink SQL对 DataSet进行过滤

   ```java
   private static final String OagOriginalData = "OagOriginalData";
   
   public static DataSet<OAGData> filterOAGData(DataSet<OAGData> oagDataSet, ExecutionEnvironment env, ParameterTool parameters) {
           /* 创建 env **/
           BatchTableEnvironment tEnv = BatchTableEnvironment.getTableEnvironment(env);
           /* 注册 DataSet **/
           tEnv.registerDataSet(OagOriginalData, oagDataSet);
           /* 获取 sql 语句 **/
           String sql = getFilterSql(parameters);
           /* 注意 sql 中 的字段名, 是Java类的字段名, 而不是数据库表中的字段名 **/
           Table tableResult = tEnv.sqlQuery(sql);
           /* 转换 **/
           DataSet<OAGData> filteredOagDataSet = tEnv.toDataSet(tableResult, OAGData.class);
           return filteredOagDataSet;
   }
   ```

6. 对2个DataSet进行join

   ```java
   //现在有2个DataSet
   DataSet<OAGData> oagDataSet;
   DataSet<SimpleOagData> thisWeekInternationalOagData;
   oagDataSet = oagDataSet.leftOuterJoin(thisWeekInternationalOagData)
   				.where(new OagFlightNoDepArrAirportKeySelector())
   				.equalTo(new SimpleOagFlightNoDepArrAirportKeySelector())
   				.with(new OagDataLeftJoinFunction())
   				.name("PUTTING FILTER LABEL ONTO OAG DATA ACCORDING TO 5 ge 1");
   //KeySelector如下
   public class OagFlightNoDepArrAirportKeySelector implements KeySelector<OAGData, String> {
       @Override
       public String getKey(OAGData oagData) throws Exception {
           return OagDataKeyUtils.getAirportComboKey(oagData);
       }
   }
   public class SimpleOagFlightNoDepArrAirportKeySelector implements KeySelector<SimpleOagData, String> {
   
       @Override
       public String getKey(SimpleOagData simpleOagData) throws Exception {
           return simpleOagData.getKey();
       }
   }
   //JoinFunction如下
   public class OagDataLeftJoinFunction implements JoinFunction<OAGData, SimpleOagData, OAGData> {
       @Override
       public OAGData join(OAGData oagData, SimpleOagData standardData) throws Exception {
           //TODO
       }
   }
   ```

   

官方参考教程：[阿里云教程](https://developer.aliyun.com/search?q=Flink入坑指南)

相关概念

1. Flink是个计算引擎，本身并不存储数据！因此需要定义Flink的上游数据源Source，以及下游数据存储Sink

2. Flink接口

   1. SQL
   2. Table API
   3. DataStream API
   4. DataSet API

3. 流SQL与批SQL

   1. 流SQL (Flink SQL)只要启动，就是一个常驻进程, 一个SQL文件，对应一个Flink作业，每来一个数据，就会产生一个结果，在数据源源不断到来时，会不停地产生结果。
   2. 批SQL每执行一次，返回一次结果，然后SQL执行结束

   比如以天为单位，实时计算当天的商品成交额

   | **ctime**               | **category_id** | **shop_id** | **item_id** | **price** |
   | :---------------------- | :-------------- | :---------- | :---------- | :-------- |
   | **2018-12-04 15:44:54** | cat_01          | shop_01     | item_01     | 10        |
   | **2018-12-04 15:45:46** | cat_02          | shop_02     | item_02     | 11.1      |
   | **2018-12-04 15:46:11** | cat_01          | shop_03     | item_03     | 12.4      |

   ```sql
   SELECT 
       date_format(ctime, '%Y%m%d') as cdate, -- 将数据从时间戳格式（2018-12-04 15:44:54），转换为date格式(20181204)
          SUM(price) AS gmv_daily
    FROM src
    GROUP BY date_format(ctime, '%Y%m%d') ; --按照天做聚合
   ```

   Flink SQL每来一条数据，会输出一个值，如果MySQL结果表中没有加主键，那将看到如下结果：

   | **cdate**    | **gmv_daily** |
   | :----------- | :------------ |
   | **20181204** | 10.0          |
   | **20181204** | 21.1          |
   | **20181204** | 33.5          |

如果mysql中把 cdate看作主键，则每来一条数据，Flink都会输出一个值，3条数据的主键相同，因此会覆盖之前的结果，最后得到的结果如下

| **cdate**    | **gmv_daily** |
| :----------- | :------------ |
| **20181204** | 33.5          |



用`group by` 和 `sum` -> 

在批SQL和流SQL中，`group by` 的行为是相同的，而 `sum`的行为有所不同

- 批：已经知道所有数据, 把每个分组的求和拿出来相加即可
- 流：数据是一条条进入系统的，并不知道全部数据，来一条加一条，产生一条结果

3条数据持续到来, flink的计算过程如下：

1. item_01: sum1=0+10
2. item_02: sum2=sum1+11.1=21.1
3. item_03: sum3=sum2+12.3=33.4

这样，每条数据sum的计算，都依赖上一条数据的计算结果，Flink在计算时，会保留这些中间结果，这就是作业的<font color="red">状态</font>(state)的一部分

只有聚合操作和`JOIN`等操作，才会保留中间结果，state的保存时间默认是36小时，如果超过超时时间，则会产生错误结果。比如把flink状态的过期时间改为5分钟，3条数据产生的时间如上面所示，但3条数据进入flink的时间，后两条数据相差超过了5分钟

1. item_01(ptime=2018-12-04 15:45:00): sum1=0+10
2. item_02(ptime=__2018-12-04 15:45:10__): sum2=sum1+11.1=21.1
3. item_03(ptime=__2018-12-04 15:52:00__): sum3=**0+12.3=12.3**
   **item_02和item_03之间超过5min，因此state中sum2的值被清掉，导致item_03到来时，sum3的值计算错误。**





注意区分2个概念：

- 数据产生的时间，流计算中的术语叫event time（事件时间）
- 数据进入Flink的事件，流计算中术语叫process time（处理时间）





一个完整的Flink SQL Job由如下三个部分组成：

- Source Operator - Soruce operator是对外部数据源的抽象, 目前Apache Flink内置了很多常用的数据源实现，比如上图提到的Kafka
- Query Operators - 查询算子主要完成如图的Query Logic，目前支持了Union，Join，Projection,Difference, Intersection以及window等大多数传统数据库支持的操作
- Sink Operator - Sink operator 是对外结果表的抽象，目前Apache Flink也内置了很多常用的结果表的抽象，比如上图提到的Kafka

![image-20200713174048311](C:/Users/iGola/AppData/Roaming/Typora/typora-user-images/image-20200713174048311.png)