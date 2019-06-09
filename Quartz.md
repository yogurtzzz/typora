# Quartz

Quartz是一个任务调度框架，用来解决如下问题：当满足某一条件时，执行某一任务

如：

1. 想当监测到系统故障时，自动发送邮件告知
2. 想每月25号自动信用卡还款
3. 想在节假日自动发送祝贺邮件等



以上问题总结起来就是：在某一个有规律的时间点干某件事，并且触发条件可以非常复杂（如，每个月最后一个工作日的17:30），复杂到需要一个专门的框架来干这事，Quartz就是来干这事的，给它一个触发条件的定义，在满足触发条件时，触发相应的Job起来干活。



## Cron表达式

| 位置 | 时间域           | 允许值 | 特殊值       |
| ---- | ---------------- | ------ | ------------ |
| 1    | 秒               | 0-59   | ,-*/         |
| 2    | 分               | 0-59   | ,-*/         |
| 3    | 时               | 0-23   | ,-*/         |
| 4    | 天（DayOfMonth） | 1-31   | ,-*/ ? L W C |
| 5    | 月               | 1-12   | ,-*/         |
| 6    | 天（DayOfWeek）  | 1-7    | ,-*/ ? L C # |
| 7    | 年（可选）       |        | ,-*/         |

特殊值详解

`*`  ： 可用于所有字段，表示对应时间域的每一个时刻，如，在`分`字段时，表示"每分钟"

`?`  :  只用于 DayOfMonth 和 DayOfWeek 中，当这两者其中之一已被指定时，需要将另一字段设为`?`，以防止冲突，因为这两个字段会相互影响

`-`  : 表示范围，如用在`分`字段时， `15-20`表示从15-20分钟，每分钟触发一次

`,`  ： 表示列表值，如在星期字段使用`MON,WED,FRI`，表示周一，周三和周五

`/`  :   x/y 表示一个等步长序列，x为起始值，y为步长，如在分钟字段中使用  `0/15` 表示0,15,30,45分的时候触发，等同于`*/15`



`L`  :   LAST，只在DayOfMonth和DayOfWeek使用，L在DayOfMonth中使用时，表示该月最后一天，如1月的31号，闰年2月的29号；L在DayOfWeek中使用时，表示星期六，等同于7，若`L`出现在星期字段，并且前面有一个数值，则表示该月的最后星期几，若星期字段是`6L`，表示该月的最后星期五



`W` : 只出现在DayOfMonth中，是对前导日期的修饰，表示离该日期最近的工作日，例如`15W`表示离该月15号最近的工作日，若该月15号是星期六，就匹配14号星期五，注意匹配日期不能跨月，如指定`1W`，若1号是星期六，结果匹配到的日期是3号星期一，而不是上个月最后那天，`W`只能指定单一日期，不能指定日期范围

​    =>  `LW`组合，表示当月最后一个工作日





`#`  :  只能在DayOfWeek中使用，  `6#3`表示当月的第3个星期五，`2#5`表示当月的第5个星期一，如果没有，则不触发



`C`  :  只能在DayOfMonth和DayOfWeek中使用，代表"Calendar"的意思，意思是计划关联的日期，在DayOfMonth中，`5C`表示当月5号之后的第一天， 在DayOfWeek中，`1C`表示星期天后的第一天



*注* : Cron表达式对特殊字符的大小写不敏感，对代表星期的英文缩写的大小写也不敏感

以下是一些例子：

| Cron表达式        | 说明                                    |
| ----------------- | --------------------------------------- |
| 0 0 12 * * ?      | 每天12点运行                            |
| 0 */5 7 * * ?     | 每天7-8点，每隔5分钟运行一次            |
| 0 */10 7,23 * * ? | 每天的7-8点，23-0点，每隔10分钟运行一次 |
| 0 0-5 14 * * ？   | 每天的14:00 到 14:05，每分钟运行一次    |
| 0 50 23 L * ？    | 每月的最后一天的23:50分运行             |
| 0 15 10 ？ * 6#3  | 每个月的第3个星期五的10:15分运行        |

 

Quartz的基本3要素

* Scheduler : 调度器，所有调度由他控制
* Trigger : 触发器
* JobDetail & Job : JobDetail定义的是任务数据，而真正的执行逻辑在Job中，为什么设计成JobDetail和Job，而不直接使用Job？ 这是因为任务有可能是并发执行的，如果Scheduler直接使用Job，就会存在同时访问一个Job实例的问题。而JobDetai和Job的方式，scheduler每次执行，都会根据JobDetail创建一个新的Job实例，这样就可以避免并发问题

以下是一个例子

```java
/** 
 * @Title:TODO  
 * @Desription:TODO
 * @Company:CSN
 * @ClassName:YogurtJob.java
 * @Author:yogurtbee
 * @CreateDate:2019-4-26   
 * @UpdateUser:yogurtbee  
 * @Version:0.1 
 *    
 */ 

package quartzDemo;

import org.quartz.Job;
import org.quartz.JobDetail;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

/** 
 * @ClassName: YogurtJob 
 * @Description: TODO 
 * @author: yogurtbee 
 * @date: 2019-4-26
 * 
 */
public class YogurtJob implements Job{

    /**
     * @param arg0
     * @throws JobExecutionException
     */
    @Override
    public void execute(JobExecutionContext ctx) throws JobExecutionException {
        // TODO Auto-generated method stub
        JobDetail jobDetail = ctx.getJobDetail();
        String name = jobDetail.getJobDataMap().getString("name");
        System.out.println("Hello quartz will not be used to " + name);
    }

}
```



```java
/** 
 * @Title:TODO  
 * @Desription:TODO
 * @Company:CSN
 * @ClassName:DemoTest.java
 * @Author:yogurtbee
 * @CreateDate:2019-4-26   
 * @UpdateUser:yogurtbee  
 * @Version:0.1 
 *    
 */ 

package quartzDemo;

import java.text.ParseException;

import org.quartz.CronScheduleBuilder;
import org.quartz.JobBuilder;
import org.quartz.JobDetail;
import org.quartz.Scheduler;
import org.quartz.SchedulerException;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.quartz.impl.StdSchedulerFactory;

/** 
 * @ClassName: DemoTest 
 * @Description: TODO 
 * @author: yogurtbee 
 * @date: 2019-4-26
 * 
 */
public class DemoTest {
    public static void main(String[] args) throws SchedulerException, ParseException, InterruptedException{
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("yogurt", "yogurtGroup")
                .withSchedule(CronScheduleBuilder.cronSchedule("* * 10 * * ?"))
                .startNow()
                .build();
        
        JobDetail jobDetail = JobBuilder.newJob(YogurtJob.class)
                .withIdentity("what","whatGroup")
                .usingJobData("name", "抠脚的大灰狼")
                .build();
        
        scheduler.scheduleJob(jobDetail, trigger);
        System.out.println("延迟3秒后启动调度器");
        scheduler.startDelayed(3);
        Thread.sleep(10000);
        scheduler.shutdown(true);
        System.out.println("任务执行完毕");
    }
}

```

执行结果如下：

![1556246190054](C:\Users\yogurtbee\AppData\Roaming\Typora\typora-user-images\1556246190054.png)



[Quartz参考链接](https://www.cnblogs.com/drift-ice/p/3817269.html)