# 1. Spring高级话题
## 1.1. Spring Aware
&emsp;在实际项目中我们可能会使用到Spring容器本身的功能资源，Spring Aware本身就是Spring设计用来框架内部使用的，它可以将完成Spring容器的资源使用。
### 1.1.1. Spring提供的Aware接口
| 接口名称 | 作用|
|--|--|
| BeanNameAware | 获取Bean容器中Bean的名称|
| BeanFactoryAware | 获取当前BeanFactory |
| ApplicationContextAware* | 当前的ApplicationContext |
| MessageSourceAware | 获取MessageSource，用来获取文本信息 |
| ApplicationEventPublisherAware | 应用事件发布器，可以发布事件 |
| ResourceLoarderAware | 获得资源加载器 |

### 1.1.2. 示例
&emsp;实现BeanNameAware
```java
@Service
public class AwareService implements BeanNameAware {
    private String beanName;

    @Override
    public void setBeanName(String s) {
        this.beanName = s;
    }

    public void outputBeanName() {
        System.out.println("Bean :" + beanName);
    }
}
```
&emsp;运行测试
```java
@ComponentScan("com.vv.spring.aware")
@Configuration
public class AwareConfig {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AwareConfig.class);

        AwareService awareService = context.getBean(AwareService.class);
        awareService.outputBeanName();
        context.close();
    }
}
```
## 1.2. 多线程
&emsp;Spring通过任务执行器(TaskExecutor)来实现多线程和并发编程。使用ThreadPoolTaskExecutor可实现一个基于线程池的TaskExecutor。实际开发中任务一般是异步调用的，所以要在配置类中通过@EnableAsync开启对异步任务的支持，并通过实际执行的Bean的方法中使用@Async注解来声明其是一个异步任务。
### 1.2.1. 示例
&emsp;配置类
```java
@Configuration
@ComponentScan("com.vv.spring.executor")
@EnableAsync
public class TaskExecutorConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(5);
        threadPoolTaskExecutor.setMaxPoolSize(10);
        threadPoolTaskExecutor.setQueueCapacity(25);
        threadPoolTaskExecutor.initialize();
        return threadPoolTaskExecutor;
    }
}
```
&emsp;任务执行类
```java
@Service
public class AsyncTaskService {

    @Async
    public void executeAsyncTask1(int i) {
        System.out.println("执行异步任务1:" + i);
    }

    @Async
    public void executeAsyncTask2(int i) {
        System.out.println("执行异步任务2:" + i);
    }
}
```
&emsp;运行
```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(TaskExecutorConfig.class);
        AsyncTaskService asyncTaskService = context.getBean(AsyncTaskService.class);
        for (int i = 0; i < 5; i++) {
            asyncTaskService.executeAsyncTask1(i);
            asyncTaskService.executeAsyncTask2(i);
        }
        context.close();
    }
}
```
&emsp;运行Main可看出结果，输出并不是顺序的。

## 1.3. 计划任务
&emsp;从Spring3.1开始，计划任务的使用变得很简单，通过@EnableScheduling开启对计划任务的支持，在要执行任务的方法上加@Scheduled，声明这是一个计划任务即可。
### 1.3.1. 示例
&emsp;定时任务执行类
```java
@Service
public class ScheduleTaskService {
    @Scheduled(fixedRate = 5000)
    public void outputTime() {
        System.out.println("每5秒一次：" + System.currentTimeMillis());
    }

    @Scheduled(cron = "0 01 11 ? * *")
    public void fixTime() {
        System.out.println("指定时间输出: " + System.currentTimeMillis());
    }
}
```
&emsp;配置类
```java
@Configuration
@ComponentScan("com.vv.spring.schedule")
@EnableScheduling
public class ScheduleConfig {
}
```
&emsp;运行
```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(ScheduleConfig.class);
    }
}
```

## 1.4. 条件注解@Conditional
&emsp;@Conditional根据满足一个特定条件创建一个特定的bean，我们可以利用这个特性进行一些自动配置。

## 1.5. 组合注解与元注解

## 1.6. @Enable*注解的工作原理
&emsp;常用的Enable注解：
| 注解 | 作用|
|--|--|
| @EnableAspectAutoProxy | 开启对AspectJ自动代理的支持|
| @EnableAsync | 开启异步方法的支持|
| @EnableAsync | 开启异步方法的支持|
| @EnableScheduling | 开启定时任务的支持|
| @EnableWebMvc | 开启Web MVC的支持|
| @EnableTransationManagement | 开启注解式事务的支持|
| @EnableCaching | 开启注解式缓存的支持|
&emsp;通过阅读这些@Enable*注解源码，可以发现它们都有一个@Import的注解，@Import是用来导入配置的，这就是说这些自动开启的实现其实是导入了一些自动配置的bean而已，它们主要分为下面三种类型
### 1.6.1. 直接导入配置类

### 1.6.2. 根据条件选择配置类
### 1.6.3. 动态注册Bean