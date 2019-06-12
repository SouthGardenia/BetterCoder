# 1. Spring概述
&emsp;Spring框架是一个轻量级的企业级开发的一站式解决方案，基于Spring可以解决Java EE开发的所有问题。Spring框架主要提供了IoC、AOP、数据访问、Web开发、消息、测试等相关技术的支持
# 2. Spring简史
## 2.1. 第一阶段
&emsp;XML配置：在Spring 1.x时代，使用Spring开发都是用xml配置的bean，开发人员需要在开发的类和配置文件之间切换，随着项目的扩大，xml配置变得很难以维护。
## 2.2. 第二阶段
&emsp;注解配置：在Spring 2.x时代，随着JDK1.5带来的注解支持，Spring提供了声明Bean的注解，大大减少了配置。
## 2.3. 第三阶段
&emsp;java配置：从Spring 3.x到现在，Spring提供了Java配置的能力，使用Java配置可以让你更理解你配置的Bean，这种配置也是现在大部分开发人员所选择的方式。
# 3. Spring基础配置
Spring框架的四大原则：<br>
1.  使用POJO进行轻量级和最小侵入式开发
2.  通过依赖注入和基于接口编程实现松耦合
3.  通过AOP和默认习惯进行声明式编程
4.  使用AOP和模版减少模式化代码  
## 3.1. 依赖注入
&emsp;依赖注入指的是容器负责创建对象和维护对象间的依赖关系，而不是通过对象本身负责自己的创建和解决自己的依赖，是IOC控制反转的一种实现方式。
### 3.1.1. 声明Bean的注解
-   @Component：组件，没有明确的角色
-   @Service：在业务逻辑层(service层)使用
-   @Repository：在数据访问层(dao层)使用
-   @Controller：在展示层使用
### 3.1.2. 注入Bean的注解
-   @Autowired：spring提供的注解
-   @Inject：JSR-330提供的注解
-   @Resource：JSR-250提供的注解
## 3.2. Java配置
&emsp;Java配置是Spring 4.x推荐的配置方式，可以完全替代xml配置，Java配置同样也是SpringBoot的配置方式。
&emsp;Java配置通过@Configuration和@Bean注解来实现
-   @Configuration声明当前方法的返回值为一个bean
-   @Bean注解在方法上，声明当前方法的返回值为一个bean
### 3.2.1. 示例
#### 3.2.1.1. 编写功能类的bean
```java
public class UserService {
    public String getUser(long userId) {
        return "user : " + userId;
    }
}
```
&emsp;代码解释：此处没有使用@Service来声明这个Bean。
#### 3.2.1.2. 使用功能类的bean
```java
class UserManageService {
    UserService userService;

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public String getUser(Long userId) {
        return userService.getUser(userId);
    }
}
```
&emsp;代码解释：此处也没用使用@Service来声明Bean，没用使用@Autowirede注入userService这个Bean
#### 3.2.1.3. 编写配置类
```java
@Configuration
class JavaConfig{
    @Bean
    public UserService userService() {
        return new UserService();
    }

    @Bean
    public UserManageService userManageService() {
        UserManageService userManageService = new UserManageService();
        userManageService.setUserService(userService());
        return userManageService;
    }
}
```
&emsp;代码解释
-   使用@Configuration注解表示这是一个配置类
-   使用@Bean注解声明的userService() 方法表示当前方法的返回值是一个Bean
-   声明UserManageService这个Bean的时候，可以直接调用UserService()方法注入Bean

#### 运行
```java
public static void main(String[] args) {
        AnnotationConfigApplicationContext configApplicationContext =
                new AnnotationConfigApplicationContext(JavaConfig.class);
        UserManageService userManageService = configApplicationContext.getBean(UserManageService.class);
        System.out.println(userManageService.getUser(1L));
        configApplicationContext.close();
    }
```
## 3.3. AOP
&emsp;Spring的Aop的存在目的是为了解耦，AOP可以让一组类共享相同的行为。在OOP中只能通过继承和实现接口来使代码，且类继承只能是单继承，阻碍更多行为添加到一组类上，AOP弥补了OOP的不足

### 实现方式
-   使用@Aspect声明是一个切面
-   使用@After、@Before、@Aroud定义建言(advice)，可直接将拦截规则(切点)
-   其中@After、@Before、@Aroud参数的拦截规则为切点(PointCut)，使用@PointCut专门定义拦截规则，然后在@After、@Before、@Aroud参数中调用

### 示例
#### 编写拦截规则的类
#### 编写使用注解被拦截的类
#### 编写使用方法规则拦截的类
#### 编写切面
-   通过@Aspect注解声明一个切面
-   通过@Component让此切面称为Spring管理的Bean
-   通过@PointCut注解声明切点
-   通过@After注解声明一个建言，并使用@PoinCut定义的切点
-   通过@Before注解声明一个建言，此建言直接使用拦截规则作为参数
#### 运行类