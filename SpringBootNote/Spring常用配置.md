# 1. Spring的常用配置
## 1.1. Bean的Scope
### 1.1.1. Scope类型
&emsp;Spring的Scope有以下几种，通过@Scope注解来实现
-   Singleton：单例，一个Spring容器中只有一个Bean实例，此为默认配置
-   Prototype：每次调用新建一个Bean的实例
-   Request：Web项目中，给每一个http request新建一个Bean实例
-   Session：Web项目中，给每一个http session新建一个Bean实例
-   GlobalSession：这个只在portal应用中有用，给每一个global session新建一个Bean实例
### 1.1.2. 用法
```java
@Service
@Scope("prototype")
public class UserService {
}
```

## 1.2. Spring EL
## 1.3. Bean的初始化和销毁
### 1.3.1. Java配置方式
&emsp;测试类
```java
@Service
public class UserService {
    public void init() {
        System.out.println("UserService init!");
    }

    public void destroy() {
        System.out.println("UserService destroy!");
    }
}
```
&emsp;配置类
```java
@Configuration
@ComponentScan("com.vv.spring.config")
public class PrePostConfig {
    @Bean(initMethod = "init", destroyMethod = "destroy")
    UserService userService(){
        return new UserService();
    }
}
```
&emsp;演示
```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext(PrePostConfig.class);
    context.getBean(UserService.class);
    context.close();
}
```
### 1.3.2. 注解方式：利用JSR-250的@PostConstruct和@PreDestroy

## 1.4. Profile

## 1.5. 事件
&emsp;Spring的事件为Bean与Bean之间通信提供了支持，当一个Bean处理完一个任务后，希望另外一个Bean知道并做相应的处理，这时就需要让另外一个Bean监听当前Bean所发送的事件
### 示例
&emsp;自定义事件
```java
public class Event extends ApplicationEvent{

    private String msg;

    public Event(String msg) {
        super(msg);
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```
&emsp;事件监听器
```java
@Component
public class Listener implements ApplicationListener<Event>{
    @Override
    public void onApplicationEvent(Event event) {
        System.out.println("接收消息：" + event.getMsg());
    }
}
```
&emsp;事件发布
```java
@Component
public class Publisher {
    @Autowired
    ApplicationContext applicationContext;

    public void publish(String msg){
        applicationContext.publishEvent(new Event(msg));
    }
}
```
&emsp;配置类
```java
@Configuration
@ComponentScan("com.vv.spring.event")
public class Config {}
```
&emsp;测试类
```java
public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);

        Publisher publisher = context.getBean(Publisher.class);
        publisher.publish("test");
        context.close();
    }
```