## SpringBoot注册Filter
在Servlet 3.0之前我们都是使用web.xml进行配置，需要增加Filter都是在web.xml增加相应的配置即可。这里我们使用的是使用Java配置来注册Filter。


## 注册Filter
1. 使用FilterRegistrationBean注册，只需要添加以下代码即可，并且他是在一个配置类里面
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170711/0f9ac62b97588134cf8eab3d1e938deg.png)


2.使用@WebFilter注册，需要在Filter类上使用该注解即可，但是需要在@Configuration类中使用Spring Boot提供的注解@ServletComponentScan扫描注册相应的Filter。

