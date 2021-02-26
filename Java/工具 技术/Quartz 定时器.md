# Quartz 定时器

## 简介

Quartz是Job scheduling（作业调度）领域的一个开源项目，Quartz既可以单独使用也可以跟spring框架整合使用，在实际开发中一般会使用后者。使用Quartz可以开发一个或者多个定时任务，每个定时任务可以单独指定执行的时间，例如每隔1小时执行一次、每个月第一天上午10点执行一次、每个月最后一天下午5点执行一次等。

### Quartz 核心概念

1.  **Job** 表示一个工作，要执行的具体内容。此接口中只有一个方法，如下：

    ```
    void execute(JobExecutionContext context) 
    ```

2.  **JobDetail** 表示一个具体的可执行的调度程序，Job 是这个可执行程调度程序所要执行的内容，另外 JobDetail 还包含了这个任务调度的方案和策略。

3.  **Trigger** 代表一个调度参数的配置，什么时候去调。

4.  **Scheduler** 代表一个调度容器，一个调度容器中可以注册多个 JobDetail 和 Trigger。当 Trigger 与 JobDetail 组合，就可以被 Scheduler 容器调度了。

### Quartz 依赖

```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
</dependency>

<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.2.1</version>
</dependency>

<!-- 需要Spring支持Context功能 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
```



## Quick Start

使用定时器的步骤：

1.  需要定义一个JobDetail
2.  需要定义一个Trigger
3.  需要定义一个simpleScheduleBuilder用来定义trigger的动作
4.  创建一个scheduler，将JobDetail与Trigger作为构造参数传入
5.  开启执行调度任务scheduler.start()
6.  结束调度任务scheduler.stop()

```java
public class QuartzTest {

    public static void main(String[] args) {
        try {
            //定义一个JobDetail
            JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)
                //定义name和group 给触发器一些属性 比如名字，组名。
                .withIdentity("job1", "group1")
                //job需要传递的内容 具体job传递参数。
                .usingJobData("name", "sdas")
                .build();
            //定义一个Trigger
            Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "group1")
                //加入 scheduler之后立刻执行 立刻启动
                .startNow()
                //定时 ，每隔1秒钟执行一次 以某种触发器触发。
                .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(1)
                              //重复执行
                              .repeatForever()).build();
            //创建scheduler
            Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
            scheduler.scheduleJob(jobDetail, trigger);
            // Scheduler只有在调用start()方法后，才会真正地触发trigger（即执行job）
            scheduler.start(); //运行一段时间后关闭
            try {
                Thread.sleep(8000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //Scheduler调用shutdown()方法时结束
            scheduler.shutdown();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }



}
```

说明：scheduler.start();启动石英调度时，会开启一个fork线程，该线程作为启动线程的守护线程。启动线程一旦结束，守护线程随即结束。



## 整合Spring

整合Spring使用石英调度来调度任务，Spring配置文件需要加入以下内容：

```xml
<!-- 注册自定义Job -->
<bean id="jobDemo" class="com.atguigu.JobDemo"></bean>

<!-- 创建JobDetail对象,作用是负责通过反射调用指定的Job，注入目标对象，注入目标方法 -->
<bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <!-- 注入目标对象 -->
    <property name="targetObject" ref="jobDemo"/>
    <!-- 注入目标方法 -->
    <property name="targetMethod" value="run"/>
</bean>

<!-- 注册一个触发器，指定任务触发的时间 -->
<bean id="myTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <!-- 注入JobDetail -->
    <property name="jobDetail" ref="jobDetail"/>
    <!-- 指定触发的时间，基于Cron表达式（0/10表示从0秒开始，每10秒执行一次） -->
    <property name="cronExpression">
        <value>0/10 * * * * ?</value>
    </property>
</bean>

<!-- 注册一个统一的调度工厂，通过这个调度工厂调度任务 -->
<bean id="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <!-- 注入多个触发器 -->
    <property name="triggers">
        <list>
            <ref bean="myTrigger"/>
        </list>
    </property>
</bean>
```

目标对象及目标方法没有限制，可以随意设置

```java
// 目标类（targetObject）
public class JobDemo {
    // 提供方法（targetMethod）
    public void run(){
        // 具体业务。。。
        System.out.println(new Date());
    }
}
```



## cron表达式（石英表达式）

### 语法：

```bash
0 0 0 * * ? [*]
```

从左到右字段的意义、可选值及特殊字符如下：

|字段 |可选值 |特殊字符 |
|-|-|-|
|秒| 0-59| , - * / |
|分 |0-59| , - * / |
|时 |0-23| , - * / |
|每月的第几天 |1-31| , - * ? / L W C |
|月份 |1-12 或者 JAN-DEC| , - * / |
|星期几，从周日开始计数 |1-7 或者 SUN-SAT| , - * ? / L C # |
|年（可选）| 1970-2099         |, - * / |

### 特殊字符说明：

**通用符号**

`*`：代表每隔一个时间单位执行一次，如果单位是秒的话，则每秒会执行一次。

`,`：代表多个指定时间单位执行，如果单位是秒，则表示在每 s1, s2, s3 秒时会触发

`-`：代表在指定间隔内，每隔时间单位执行一次，如秒为 15-25 ，则会在 15s 到 25s 之间，每隔一秒执行一次

`/`：代表每个 / 后的时间单位执行一次，如秒为 /10 ，则表示每隔十秒执行一次

**星期几**

`#`：指定该月中具体的周数，用法为 `W#D`，其中W为week，D为day，表示该月的第W周的星期D，

**每月的第几天 / 星期几**

`?`：两者之一必须为`?`，防止冲突。

`L`：最后一天触发

### 经典案例

| cron表达式 | 说明 |
| ---------- | ---- |
|`"30 * * * * ?"` |每半分钟触发任务|
|`"30 10 * * * ?"` |每小时的10分30秒触发任务|
|`"30 10 1 * * ?"` |每天1点10分30秒触发任务|
|`"30 10 1 20 * ?"` |每月20号1点10分30秒触发任务|
|`"30 10 1 20 10 ? *"` |每年10月20号1点10分30秒触发任务|
|`"30 10 1 20 10 ? 2011"` |2011年10月20号1点10分30秒触发任务|
|`"30 10 1 ? 10 * 2011"` |2011年10月每天1点10分30秒触发任务|
|`"30 10 1 ? 10 SUN 2011"` |2011年10月每周日1点10分30秒触发任务|
|`"15,30,45 * * * * ?"` |每15秒，30秒，45秒时触发任务|
|`"15-45 * * * * ?"` |15到45秒内，每秒都触发任务|
|`"15/5 * * * * ?"` |每分钟的每15秒开始触发，每隔5秒触发一次|
|`"15-30/5 * * * * ?"` |每分钟的15秒到30秒之间开始触发，每隔5秒触发一次|
|`"0 0/3 * * * ?"` |每小时的第0分0秒开始，每三分钟触发一次|
|`"0 15 10 ? * MON-FRI"` |星期一到星期五的10点15分0秒触发任务|
|`"0 15 10 L * ?"` |每个月最后一天的10点15分0秒触发任务|
|`"0 15 10 LW * ?"` |每个月最后一个工作日的10点15分0秒触发任务|
|`"0 15 10 ? * 5L"` |每个月最后一个星期四的10点15分0秒触发任务|
|`"0 15 10 ? * 5#3"` |每个月第三周的星期四的10点15分0秒触发任务|