# log4j配置日志输出

log4j支持两种配置文件格式，一种是XML，一种是properties，这里只记录properties方式

1.首先在用到了log4j的java文件的`src`目录级别下创建一个`log4j.properties`的文件（貌似会被自动装载的样子）

2.在`log4j.properties`文件中进行log4j相关配置，说明如下：



## 1. rootLogger

配置方式：

```properties
log4j.rootLogger=[level],[appendName1],[appendName2],......
```



其中

```properties
# level 是日志记录的优先级，默认有（优先级从低到高）： 
# ALL , (DEBUG , INFO , WARN , ERROR), FATAL , OFF
# 建议只使用4个级别，如上面小括号内
# 注意： () parenthese   [] brackets  {} braces
# DEBUG , INFO , WARN , ERROR
# 若设置rootLogger的level为INFO，那么会输出 INFO,WARN,ERROR级别的日志
# 若设置为WARN，那么会输出WARN,ERROR级别的日志
# 可以这样记忆：设置了level后，该level之后的level的日志将会被输出，而之前level的不会输出

# appendName是自定义的，不是固定句式

# 如
log4j.rootLogger = INFO,haha,test,stdout
```



## 2. Appender

配置日志信息的输出位置，格式如下

```properties
log4j.appender.appenderName1 = [log4j提供的appender类]
log4j.appender.appenderName1.属性名1 = [属性值1]
log4j.appender.appenderName1.属性名2 = [属性值2]
........
```

log4j提供的appender类有以下5种

```properties
#控制台
org.apache.log4j.ConsoleAppender
#文件
org.apache.log4j.FileAppender
#每天产生一个日志文件
org.apache.log4j.DailyRollingFileAppender
#文件大小到达指定尺寸后，产生一个新文件
org.apache.log4j.RollingFileAppender
#将日志信息以流的格式发送到任意指定的地方
org.apache.log4j.WriterAppender

```

举例，配置实现将日志输出到控制台

```properties
log4j.appender.yogurt = org.apache.log4j.ConsoleAppender
log4j.appender.yogurt.Threshold = ERROR
#下面默认情况是System.out（黑色）,而System.err打印出来是红色
log4j.appender.yogurt.Target = System.err
#下面默认值为true，即所有消息会被立即输出
log4j.appender.yogurt.ImmediateFlush = true
log4j.appender.yogurt.layout = org.apache.log4j.PatternLayout
log4j.appender.yogurt.layout.ConversionPattern = [%5p] %d{yyyy-MM-dd HH:mm:ss} %l - %m%n
```



举例，配置实现将日志输出到指定文件

```properties
log4j.appender.A = org.apache.log4j.RollingFileAppender
log4j.appender.A.encoding = UTF-8
log4j.appender.A.Threshold = INFO
log4j.appender.A.Append = true
log4j.appender.A.File = ${catalina.home}/log/mylog.log
log4j.appender.A.layout = org.apache.log4j.PatternLayout
log4j.appender.A.layout.ConversionPattern = %d{yyyy-MM-dd HH:mm:ss} %-5p - %m%n
# 下面指定日志文件的大小，在日志文件达到该大小时，会自动滚动，即将新日志移动到mylog.log.1文件
log4j.appender.A.MaxFileSize = 100KB
# 下面指定可产生的滚动文件的最大数量，即最多产生50个滚动文件，即最多到mylog.log.50
log4j.appender.A.MaxBackupIndex = 50
```



## 3. layout

日志信息的格式，配置格式如下

```properties
log4j.appender.appenderName1.layout = [log4j提供的layout类]
log4j.appender.appenderName1.layout.属性名 = [属性值]
....
```

log4j提供的layout类如下

```properties
# 可自定义地指定日志格式，最常用
org.apache.log4j.PatternLayout
# 以HTML表格形式布局
org.apache.log4j.HTMLLayout
# 包含日志信息的级别和信息字符串
org.apache.log4j.SimpleLayout
# 包含日志产生的时间，线程，类别邓信息
org.apache.log4j.TTCCLayout
```

HTMLLayout

`LocationInfo = true`  输出java文件名称和行号，默认值是false

`Title = my log file` 默认值是 Log4J Log Messages



对于PatternLayout，需要指定如何格式化信息

```properties
log4j.appender.appenderName1.layout.ConversionLayout = %-5p %d{yyyy-MM-dd HH:mm:ss} %c %m%n
# 类似与用c语言中的printf来格式化日志信息，相关参数说明如下
%p 日志优先级，即 DEBUG,INFO,ERROR等
%d 产生日志的时间点
%c 产生日志的类，通常是类全名
%t 产生日志的线程名
%l 日志发生的位置，包括类名，方法名，行数等，相当于 %C.%M(%F:%L)
# e.g. Testlog4j.main(Testlog4j.java:10)
%x 当前线程相关联的NDC（嵌套诊断环境），尤其用在java servlets中
%% 输出一个%号
%F 输出产生日志的文件名称
%L 输入代码中的行号
%m 输出日志的具体信息
%n 换行符

# 可在%与模式字符之间加上修饰符，来控制最小最大宽度，和对齐方式
%5p 输出日志优先级，最小宽度为5，若宽度小于5，默认右对齐
%-5p 若宽度小于5，-符号指定左对齐
%.10c 指定输出类名，最大宽度为10，如果类名宽度大于10，则将左边多出的字符截去，若宽度小于10，也不会出现空格
```



举例，HTMLLayout

```properties
log4j.rootLogger=INFO,stdout,html
log4j.appender.html = org.apache.log4j.RollingFileAppender
log4j.appender.html.layout = org.apache.log4j.HTMLLayout
log4j.appender.html.Threshold = INFO
log4j.appender.html.Append = true
log4j.appender.html.layout.LocationInfo = true
log4j.appender.html.layout.Title = my log file HTML
log4j.appender.html.File = htmlLog.html
log4j.appender.html.MaxFileSize = 10KB
log4j.appender.html.MaxBackupIndex = 3
```

测试：

![1551927541525](C:\Users\yogurtbee\AppData\Roaming\Typora\typora-user-images\1551927541525.png)

[参考链接](https://blog.csdn.net/eagleuniversityeye/article/details/80582140)