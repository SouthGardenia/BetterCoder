# SpringBoot核心
## 基本配置
### 入口类
### 关闭特定的自动配置
### 定制banner
### SpringBoot的配置文件

## SpringBoot运行原理
### 运作原理
&emsp;关于SpringBoot的运作原理，还是要先看@SpringBootAppplication注解上来，这个注解是一个组合注解，它的核心功能是@EnableautoConfiguration注解提供的,其源码如下
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

```
&emsp;这里的关键点是@Import注解导入的配置功能