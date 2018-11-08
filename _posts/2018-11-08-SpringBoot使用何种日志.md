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
这里有一个细节要注意，根据代码所示，它第一个会去查找org.springframework.boot.logging.logback.LogbackLoggingSystem，但如果你的类路径下没有logback实现，而只有log4j实现的话，是会报错的，报错的原因是下面这段代码引起的，也许这就是一个坑吧  
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170724/e3d3f5f482f088d573fc2cf230b4ee61.png)
