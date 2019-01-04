## 通过sleep循环实现定时

创建一个thread，然后让它在while循环里一直运行着，通过sleep方法来达到定时任务的效果

```java
/**
 * 创建一个thread，然后让它在while循环里一直运行着，通过sleep方法来达到定时任务的效果
 */
public class JavaTimerTest {
    public static void main(String[] args){
        final long timeInterval = 1000;
        Runnable runnable=new Runnable() {
            public void run() {
                while (true){
                    System.out.println("hello,task test...");
                    try {
                        Thread.sleep(timeInterval);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
    }
}
```

## 用Timer和TimerTask

用Timer和TimerTask与第一种方法相比有如下好处：

 *  java.util.Timer;
 *  java.util.TimerTask;

    1. 当启动和去取消任务时可以控制
    2. 第一次执行任务时可以指定你想要的delay时间


```java
/**
 * 用Timer和TimerTask与第一种方法相比有如下好处：
 *  java.util.Timer;
 *  java.util.TimerTask;

        1. 当启动和去取消任务时可以控制

        2. 第一次执行任务时可以指定你想要的delay时间
 */
public class JavaTimerTest2 {
    public static void main(String[] args){
        TimerTask timerTask=new TimerTask() {
            @Override
            public void run() {
                System.out.println("hello, task test2...");
            }
        };

        Timer timer = new Timer();
        long delay = 0;
        long intevalPeriod = 1*1000;
        timer.scheduleAtFixedRate(timerTask,delay,intevalPeriod);
    }
}
```

## ScheduledExecutorService

用ScheduledExecutorService是从的java.util.concurrent里，做为并发工具类被引进的，这是最理想的定时任务实现方式，相比于上两个方法，它有以下好处:

 *     1.相比于Timer的单线程，它是通过线程池的方式来执行任务的
 *     2.可以很灵活的去设定第一次执行任务delay时间
 *     3.提供了良好的约定，以便设定执行的时间间隔

```java
/**
 * 用ScheduledExecutorService是从的java.util.concurrent里，做为并发工具类被引进的，这是最理想的定时任务实现方式，相比于上两个方法，它有以下好处:
 *     1.相比于Timer的单线程，它是通过线程池的方式来执行任务的
 *     2.可以很灵活的去设定第一次执行任务delay时间
 *     3.提供了良好的约定，以便设定执行的时间间隔
 */
public class JavaTimerTest03 {
    public static void main(String[] args){
        Runnable runnable=new Runnable() {
            public void run() {
                System.out.println("hello,task test3...");
            }
        };
        ScheduledExecutorService service= Executors.newSingleThreadScheduledExecutor();
        service.scheduleAtFixedRate(runnable, 0, 2, TimeUnit.SECONDS);
    }
}
```

## Spring Quartz方式一

### java程序

```java
public class QuartzTask {
    private static int i;
    //    private Lock lock = new ReentrantLock();
    //通过锁机制进行多线程并发操作
    private static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    static {
        i=3;
    }
    public void  work(){
        ReentrantReadWriteLock.WriteLock writeLock=lock.writeLock();
        try {
            writeLock.lock();
            System.out.println(Thread.currentThread().getId() + " - " + Thread.currentThread().getName() + " quartz task runing .. " + i++);
        }finally {
            writeLock.unlock();
        }
    }
}
```

### spring配置

```xml
<!--spring-context.xml配置-->
<!--要调用的工作类-->
<bean id="quartzTask" class="com.ibingbo.task.QuartzTask"></bean>
<!--定义一个作业，指定具体工作对象及工作对象的方法-->
<bean id="jobtask" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <!--调用的类-->
    <property name="targetObject">
        <ref bean="quartzTask"/>
    </property>
    <!--调用的方法-->
    <property name="targetMethod">
        <value>work</value>
    </property>
</bean>
<!--定义触发时间-->
<bean id="doTime" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <!--绑定的作业任务-->
    <property name="jobDetail">
        <ref bean="jobtask"/>
    </property>
    <!--运行周期-->
    <property name="cronExpression">
        <value>*/1 * * * * ?</value>
    </property>
</bean>
```

## Spring Quartz方式二

在Spring中声明并且配置作业调度的触发方式

### java程序

```java
/**
 * 使用 Quartz
 * 在Spring中声明并且配置作业调度的触发方式
 */
public class SpringQuartzTimer extends QuartzJobBean{

    private int timeout;
    private static int i=0;

    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }

    protected void executeInternal(org.quartz.JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println(Thread.currentThread().getId() + " - "+Thread.currentThread().getName() +" spring task run..." + i++);
    }

}
```

### spring配置

```xml
<bean name="job1" class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
    <property name="jobClass" value="com.ibingbo.task.SpringQuartzTimer"/>
    <property name="jobDataAsMap">
        <map>
            <entry key="timeout" value="0"/>
        </map>
    </property>
</bean>

<!--简单按照一定频度调用任务，如每隔30分钟运行一次-->
<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
    <property name="jobDetail" ref="job1"/>
    <property name="startDelay" value="0"/><!-- 调度工厂实例化后，经过0秒开始执行调度 -->
    <property name="repeatInterval" value="2000"/><!-- 每2秒调度一次 -->
</bean>

<!--支持到指定时间运行一次-->
<bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="job1"/>
    <property name="cronExpression" value="0 * * * * ?"/>
</bean>
```

## Spring Task

### java程序

```java
@Service
public class SpringTask {

    public void job1(){
        System.out.println(Thread.currentThread().getId() + " - " + Thread.currentThread().getName() + " spring task job1 runing .. ");
    }
}
```

### spring配置

```xml
<!--********************************spring task任务的配置方式*****************************-->
<task:scheduled-tasks>
    <task:scheduled ref="springTask" method="job1" cron="5 * * * * ?"/>
</task:scheduled-tasks>
<!--********************************spring task任务的配置方式*****************************-->
```

## Spring Task注解方式

### 程序中应用@Scheduled注解

```java
/**
 * spring task 注解方式
 * 需要在xml里开启注解<task:annotation-driven scheduler="qbScheduler" mode="proxy"/>
 */
@Component("springTaskAnnotation")
public class SpringTaskAnnotation {
    private final static Logger logger = LoggerFactory.getLogger(SpringTaskAnnotation.class);
    private static Integer i;
    private static Boolean loaded = false;
    private static LinkedList list = new LinkedList();

    private final static ReentrantReadWriteLock rw = new ReentrantReadWriteLock();
    private static LinkedList list2 = new LinkedList();
    private static Boolean loaded2 = false;

    private final static ReentrantReadWriteLock rw3 = new ReentrantReadWriteLock();
    private static LinkedList list3 = new LinkedList();
    private static Boolean loaded3 = false;

    static {
        i = 3;

    }

    //@Scheduled(cron = "*/5 * * * * ?")
    public void work() {
        synchronized (i) {
            System.out.println(Thread.currentThread().getId() + " - " + Thread.currentThread().getName() + " spring task annotation runing .. " + i++);
        }
    }

    {

    }

    //@Scheduled(cron = "0 0/1 * * * ?")
    public void work2() {

        synchronized (loaded) {
            if (!loaded) {
                for (int i = 0; i < 1000; i++) {
                    list.add(i);
                }
                loaded = true;
            }
        }
        while (true) {
            synchronized (list) {
                Object o = list.poll();
                if (o == null) {
                    loaded = false;
                    break;

                }
                System.out.println(Thread.currentThread().getId() + " - " + Thread.currentThread().getName() + " spring task annotation runing .. " + o.toString() + " size:" + list.size());
            }
        }
    }

    @Scheduled(cron = "0 0/2 * * * ?")
    public void work3() {
        rw.writeLock().lock();
        try {
            if (!loaded2) {
                for (int i = 0; i < 1000; i++) {
                    list2.add(i);
                }
                loaded2 = true;
            }
        } finally {
            rw.writeLock().unlock();
        }

        while (true) {
            rw.writeLock().lock();
            try {
                Object o = list2.poll();
                if (o == null) {
                    loaded2 = false;
                    break;
                }
                System.out.println(Thread.currentThread().getId() + " - " + Thread.currentThread().getName() + "  work 2.. " + o.toString() + " size:" + list2.size());

            } finally {
                rw.writeLock().unlock();
            }
        }
    }

    @Scheduled(cron = "0 0/3 * * * ?")
    public void work4() {
        rw3.writeLock().lock();
        try {
            if (!loaded3) {
                for (int i = 0; i < 1000; i++) {
                    list3.add(1000 + i);
                }
                loaded3 = true;
            }
        } finally {
            rw3.writeLock().unlock();
        }

        while (true) {
            rw3.writeLock().lock();
            try {
                Object o = list3.poll();
                if (o == null) {
                    loaded3 = false;
                    break;
                }
                System.out.println(Thread.currentThread().getId() + " - " + Thread.currentThread().getName() + "  work 3.. " + o.toString() + " size:" + list3.size());

            } finally {
                rw3.writeLock().unlock();
            }
        }
    }
}
```

### spring配置,开启任务注解方式,指定任务调度池，及调用方式为代理

```xml
<task:annotation-driven scheduler="poolTaskScheduler" mode="proxy"/>
<!--任务线程池-->
<task:scheduler id="poolTaskScheduler" pool-size="10"/>
```
