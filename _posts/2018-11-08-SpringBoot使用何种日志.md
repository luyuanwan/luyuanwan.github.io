## 结论
SpringBoot按照以下的顺序查找日志实现
- org.springframework.boot.logging.logback.LogbackLoggingSystem（logback）
- org.springframework.boot.logging.log4j2.Log4J2LoggingSystem（log4j）
- org.springframework.boot.logging.java.JavaLoggingSystem（Java自带日志实现）

## 代码分析
首先看一下SpringBoot中的日志部分的代码结构，如下图所示  
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170724/e3d3f5f482f088d573fc2cf230b4ee57.png)

SpringBoot为了更好地管理日志，特别写了一个LoggingSystem抽象类，如下图所示，可以看到，在静态块中添加了3种日志实现，并赋值给SYSTEMS变量  
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170724/e3d3f5f482f088d573fc2cf230b4ee58.png)

## 问题：SpringBoot是如何接入日志的呢
SpringBoot借助事件来接入日志的，在Spring中当完成某些事情的时候会发送event，当接收到event时你就可以做一些事情，如下图所示，这是LoggingApplactionListener类，它监听了Spring事件，并且在系统启动的时候处理日志，处理的过程就是查找到当前第一个可用的日志实现，并初始化它  
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170724/e3d3f5f482f088d573fc2cf230b4ee59.png)

我们来看看LoggingSystem.get函数都干了些什么，如下图所示  
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170724/e3d3f5f482f088d573fc2cf230b4ee60.png)

它首先取系统信息，如果配置了系统信息，则使用它作为日志实现，否则就在SYSTEMS中查找第一个在类路径上可以找到的日志实现

### 注意事项
下面这段代码会引起logback异常，有时间去分析一下原因:)
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170724/e3d3f5f482f088d573fc2cf230b4ee61.png)

### 如何切换日志
有2种办法切换日志实现
1. 系统变量中写入你要的日志实现信息，比如下面这样（同时你要在类路径下存在相应的实现包）

```java
//logback实现
System.setProperty("org.springframework.boot.logging","org.springframework.boot.logging.logback.LogbackLoggingSystem");
//log4j实现
System.setProperty("org.springframework.boot.logging","org.springframework.boot.logging.log4j2.Log4J2LoggingSystem");
//Java自带实现
System.setProperty("org.springframework.boot.logging","org.springframework.boot.logging.java.JavaLoggingSystem");
```

2. 在类路径下需要存在以下以类，就能按顺序选择相应实现  
存在ch.qos.logback.core.Appender，则选择logback实现  
存在org.apache.logging.log4j.core.impl.Log4jContextFactory，则选择log4j实现  
存在java.util.logging.LogManager，则选择Java自带实现  
