---
title: Quartz学习
author: Juntech
top: true
cover: false
toc: true
mathjax: false
summary: Quartz
categories: Quartz
tags: Quartz
keywords: Quartz
abbrlink: '3808e305'
date: 2019-08-30 12:42:09
password:
copyright: true
---

# Quartz学习

## 1、引入依赖

去到<https://mvnrepository.com/search?q=quartz>

寻找Quartz,Quartz-jobs

```xml
<!-- https://mvnrepository.com/artifact/org.quartz-scheduler/quartz -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.quartz-scheduler/quartz-jobs -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.3.1</version>
</dependency>

```

目前只需要这两个包就行了，本节只是对quartz他的流程进行依次简单的运行。

## 2、创建job任务

HelloJob.java

```java
package top.juntech.job;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

import java.text.SimpleDateFormat;
import java.util.Date;


public class HelloJob implements Job {
//   重写execute方法
//    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
//        输出当前时间
        Date date = new Date();
//        格式化时间
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//        时间
        String formatdate = simpleDateFormat.format(date);
        System.out.println("正在备份数据库，备份数据库的日期是： "+formatdate);
    }
}
```

该任务集成Job接口，重写execute方法，写入自己要进行的任务。比如在这，我们就直接打印一个时间作为本次训练的任务。

## 3、创建main程序

1. 使用任务调度会产生io异常,我们直接在main程序上抛出大异常 throw Exception

2. 从工厂中创建任务调度器实例  

   ```java
   //        调度器 (StdShedulerFactory实例）
           Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
   ```

3. 创建任务实例

   ```java
   //        任务实例 jobDetail
           JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)  //加载人物类
                   .withIdentity("job1", "group1")  //param1: job1是自定义名字，param2:自定义组
                   .build();

   ```

4. 创建触发器实例

   ```java
   //        触发器
           SimpleTrigger trigger = TriggerBuilder.newTrigger()
                   .withIdentity("trigger1", "group1")  //param1:自定义，param2:触发器组名
                   .withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(5))  //每5s执行一次
                   .build();

   ```

5. 让调度器实例关联触发器和任务并启动

   ```java
   //        让调度器关联任务和触发器
           scheduler.scheduleJob(jobDetail,trigger);
   //        启动scheduler
           scheduler.start();
   ```

   控制台打印：

   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:03
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:08
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:13
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:18
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:23
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:28
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:33
   正在备份数据库，备份数据库的日期是： 2019-08-30 11:43:38

   可以看到每5秒执行一次job

## 4、JobDataMap及有无状态的Job

使用jobDatamap : usingJobData(Map<key,val>)

```java
//        任务实例 jobDetail
        JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)  //加载人物类
                .withIdentity("job1", "group1")
                .usingJobData("jobsays","haha")
                .usingJobData("john","john")
                .usingJobData("count",0)
                .build();
```

在任务job中调用数据：

1.count计数

```java
//    count值
    private Integer count;

    public void setCount(Integer count) {
        this.count = count;
    }
```

2.获取jobDatamap的值：

```java
    获取jobdetail数据
        JobKey jobKey = jobExecutionContext.getJobDetail().getKey();
        JobDataMap jobDataMap = jobExecutionContext.getJobDetail().getJobDataMap();
        String jobsays = jobDataMap.getString("jobsays");
        count = jobDataMap.getInt("count");
        ++count;
        jobExecutionContext.getJobDetail().getJobDataMap().put("count",count);
        System.out.println("count : "+count);
```

控制台打印：

count : 1
正在备份数据库，备份数据库的日期是： 2019-08-30 14:28:24,jobsys : haha
count : 2
正在备份数据库，备份数据库的日期是： 2019-08-30 14:28:29,jobsys : haha
count : 3
正在备份数据库，备份数据库的日期是： 2019-08-30 14:28:34,jobsys : haha
count : 4
正在备份数据库，备份数据库的日期是： 2019-08-30 14:28:39,jobsys : haha

## 5、simpleTrigger触发器

指定时间开始触发，不重复：

```java
    SimpleTrigger trigger = (SimpleTrigger) newTrigger() 
        .withIdentity("trigger1", "group1")
        .startAt(myStartTime)                     // some Date 
        .forJob("job1", "group1")                 // identify job with name, group strings
        .build();
```

指定时间触发，每隔10秒执行一次，重复10次：

```java
    trigger = newTrigger()
        .withIdentity("trigger3", "group1")
        .startAt(myTimeToStartFiring)  // if a start time is not given (if this line were omitted), "now" is implied
        .withSchedule(simpleSchedule()
            .withIntervalInSeconds(10)
            .withRepeatCount(10)) // note that 10 repeats will give a total of 11 firings
        .forJob(myJob) // identify job with handle to its JobDetail itself                   
        .build();
```

5分钟以后开始触发，仅执行一次：

```java
    trigger = (SimpleTrigger) newTrigger() 
        .withIdentity("trigger5", "group1")
        .startAt(futureDate(5, IntervalUnit.MINUTE)) // use DateBuilder to create a date in the future
        .forJob(myJobKey) // identify job with its JobKey
        .build();
```

立即触发，每个5分钟执行一次，直到22:00：

```java
    trigger = newTrigger()
        .withIdentity("trigger7", "group1")
        .withSchedule(simpleSchedule()
            .withIntervalInMinutes(5)
            .repeatForever())
        .endAt(dateOf(22, 0, 0))
        .build();
```

建立一个触发器，将在下一个小时的整点触发，然后每2小时重复一次：

```java
    trigger = newTrigger()
        .withIdentity("trigger8") // because group is not specified, "trigger8" will be in the default group
        .startAt(evenHourDate(null)) // get the next even-hour (minutes and seconds zero ("00:00"))
        .withSchedule(simpleSchedule()
            .withIntervalInHours(2)
            .repeatForever())
        // note that in this example, 'forJob(..)' is not called which is valid 
        // if the trigger is passed to the scheduler along with the job  
        .build();

    scheduler.scheduleJob(trigger, job);
```

请查阅TriggerBuilder和SimpleScheduleBuilder提供的方法，以便对上述示例中未提到的选项有所了解。

```java
TriggerBuilder(以及Quartz的其它builder)会为那些没有被显式设置的属性选择合理的默认值。比如：如果你没有调用withIdentity(..)方法，TriggerBuilder会为trigger生成一个随机的名称；如果没有调用startAt(..)方法，则默认使用当前时间，即trigger立即生效。
```

**SimpleTrigger Misfire策略**

SimpleTrigger有几个misfire相关的策略，告诉quartz当misfire发生的时候应该如何处理。(Misfire策略参考[教程四：Trigger介绍](http://nkcoder.github.io/blog/20140716/quartz-tutorial-04-trigger/))。这些策略以常量的形式在SimpleTrigger中定义(JavaDoc中介绍了它们的功能)。这些策略包括：

SimpleTrigger的Misfire策略常量：

```java
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFIRE_INSTRUCTION_FIRE_NOW
MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT
MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT
```

回顾一下，所有的trigger都有一个Trigger.MISFIRE_INSTRUCTION_SMART_POLICY策略可以使用，该策略也是所有trigger的默认策略。

如果使用smart policy，SimpleTrigger会根据实例的配置及状态，在所有MISFIRE策略中动态选择一种Misfire策略。SimpleTrigger.updateAfterMisfire()的JavaDoc中解释了该动态行为的具体细节。

在使用SimpleTrigger构造trigger时，misfire策略作为基本调度(simple schedule)的一部分进行配置(通过SimpleSchedulerBuilder设置)：

```java
    trigger = newTrigger()
        .withIdentity("trigger7", "group1")
        .withSchedule(simpleSchedule()
            .withIntervalInMinutes(5)
            .repeatForever()
            .withMisfireHandlingInstructionNextWithExistingCount())
        .build();
```

## 6、CronTrigger触发器

建立一个触发器，每隔一分钟，每天上午8点至下午5点之间：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
    .forJob("myJob", "group1")
    .build();
```

建立一个触发器，将在上午10:42每天发射：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(dailyAtHourAndMinute(10, 42))
    .forJob(myJobKey)
    .build();
```

或者：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 42 10 * * ?"))
    .forJob(myJobKey)
    .build();
```

建立一个触发器，将在星期三上午10:42在TimeZone（系统默认值）之外触发：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42))
    .forJob(myJobKey)
    .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
    .build();
```

或者：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 42 10 ? * WED"))
    .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
    .forJob(myJobKey)
    .build();
```

### CronTrigger Misfire说明

以下说明可以用于通知Quartz当CronTrigger发生失火时应该做什么。（本教程“更多关于触发器”部分引入了失火情况）。这些指令定义为CronTrigger本身的常量（包括描述其行为的JavaDoc）。说明包括：

CronTrigger的Misfire指令常数

```
MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY
MISFIRE_INSTRUCTION_DO_NOTHING
MISFIRE_INSTRUCTION_FIRE_NOW
```

所有触发器还具有可用的Trigger.MISFIRE_INSTRUCTION_SMART_POLICY指令，并且该指令也是所有触发器类型的默认值。“智能策略”指令由CronTrigger解释为MISFIRE_INSTRUCTION_FIRE_NOW。CronTrigger.updateAfterMisfire（）方法的JavaDoc解释了此行为的确切细节。

在构建CronTriggers时，您可以将misfire指令指定为简单计划的一部分（通过CronSchedulerBuilder）：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?")
        ..withMisfireHandlingInstructionFireAndProceed())
    .forJob("myJob", "group1")
    .build();
```



## 7、参考资料

w3school: <https://www.w3cschool.cn/quartz_doc/quartz_doc-1xbu2clr.html>