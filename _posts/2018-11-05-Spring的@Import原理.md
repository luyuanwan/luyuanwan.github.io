---
layout:     post
title:      Spring的@Import原理
date:       2018-11-05
author:     大雁小鱼
catalog:    true
tags:
    - Spring
---

# 前言
在使用Spring的时候会使用@Import注解，这个注解是怎么起作用的，它有什么作用呢，今天我就来聊聊这个注解的前世今生。

# @Import使用方法
@Import的使用方法如下
```java
@Import(DynamicDataSourceRegister.class)
public class DBConfigure {

}

```
这段代码的意思是在系统启动的时候，请导入类DynamicDataSourceRegister

# @Import作用
@Import的作用有些人可能会误解，以为是将这个类作为一个Bean放入spring容器中，其实不然，这个类不会是一个Bean的，Spring会执行这个类，然后这个类会生成一些bean，简单地说就是这是一个Bean工厂，所以如果你Import了一个普通类，是没啥卵用的。

@Import(XX.class)  ->  XX类是一个工厂类，它会生成好多Bean放入Spring容器中

# @Import是如何起作用的呢
AbatractApplicationContext类中有一个大家熟悉的refresh()方法，如下所示

```java
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			// 初始化配置准备刷新，验证环境变量中的一些必选参数
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			// 告诉继承类销毁内部的factory，创建新的factory实例
			// 如果是XML，在这里会读取XML并生成BeanDefinition
			// 生成可配置的bean工厂
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			// 初始化beanfactory的基本信息，包括classloader,environment
			// 这里会添加一些BeanPostProcessor
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 后处理 bean工厂
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				// 调用BeanFactoryPostProcessors 在这里处理 @Import @Configureation这些生成bean的注解信息
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				//注册BEAN的POST处理器,在调用之前，工厂里面已经有所有BEAN了
				//这里注册了BeanPostProcessor，包括AutowiredAnnotationBeanPostProcessor
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				//实例化bean并注入依赖,这里会生成non-lazy singleton beans
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				logger.warn("Exception encountered during context initialization - cancelling refresh attempt", ex);

				// Destroy already created singletons to avoid dangling resources.
				// 销毁已经创建的单例
				destroyBeans();

				// Reset 'active' flag.
				//将状态置无效，标识初始化失败
				cancelRefresh(ex);

				// Propagate exception to caller.
				//将异常传播到调用者
				throw ex;
			}
		}
	}
```
第一步：  
在这个方法中，有一个obtainFreshBeanFactory方法，在这个obtainFreshBeanFactory方法中会扫描所有的Bean，于是所有的类定义都会被扫描到，包括含有@Import注解的类

第二步：  
在这个方法中，有一个invokeBeanFactoryPostProcessors,在这个方法中，会对所有的带有@Import注解的类进行处理，代码如下（由于函数很长，我只截取了一部分代码放在这里，类是ConfigurationClassParser）

```java
//实例化了一个带有@Import注解的类
ImportBeanDefinitionRegistrar registrar =
    BeanUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class);
//调用TA的Aware方法
invokeAwareMethods(registrar);
configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
````
从这段代码中，我们可以清楚地看到，它实例化了一个带有@Import注解的类，且要求实现了ImportBeanDefinitionRegistrar接口，并且最后会调用ImportBeanDefinitionRegistrar.registerBeanDefinitions方法注册它生产出来的bean到Spring容器中

这就是@Import的完整功能
