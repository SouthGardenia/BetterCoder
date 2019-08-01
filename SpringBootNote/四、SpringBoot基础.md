# SpringBoot基础    
## SpringBoot核心功能
### 独立运行的Spring项目
&emsp;SpringBoot可以以jar包的形式运行，运行一个Springboot项目只需通过 java -jar
### 内嵌servlet容器
&emsp;SpringBoot可以选择内嵌的Tomcat、Jetty、Undertow，这样我们无需以war包形式部署项目
### 提供starter简化maven配置
&emsp;Spring提供了一系列的starter pom来简化Maven的依赖加载
### 自动配置Spring
### 准生产的应用监控
&emsp;SpringBoot提供基于http、ssh、telnet对运行时的项目进行监控
### 无代码生成和xml配置
&emsp;Spring4.x提倡使用Java配置和注解配置组合，而SpringBoot不需要任何xml配置即可实现Spring所有配置


## SpringBoot优点
-   快速构建项目
-   对主流开发框架的无配置集成
-   项目可独立运行，无须外部依赖Servlet容器
-   提供运行时的应用监控
-   极大得提高了开发、部署效率

## 快速搭建SpringBoot项目