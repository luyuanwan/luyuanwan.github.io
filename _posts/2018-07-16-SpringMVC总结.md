## SpringMVC总结

SpringMVC是一款被许多大小型公司使用的热门MVC框架，本人不才，今日仅就我所知道的内容做一些整理，还望各位大大不吝赐教。

### SpringMVC是什么
SpringMVC的核心是一个Servlet，是一个易于开发者使用的Servlet开发框架，易于使用的地方主要体现在：  
1、添加一个URL的功能非常方便，不再像很早之前使用WebWork(Struct1的前身)一样需要频繁修改web.xml配置；  
2、获取前端传递的参数很方便，只需要加上一些合适的注解即可(Struct2获取参数较为不便)；  
3、一个URL功能可以被映射到一个方法上，而不用像struct2一个功能只能映射到一个类；  
4、可以依托Spring强大的IOC，获取Bean非常方便，结偶能力大大提升。  
SpringMVC的核心类是DispatcherServlet类，其类结构如下图所示
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170930/005e48ef3c6b700c411219ebc399e05e.png)

### SpringMVC怎么做
SpringMVC的强大来源于它背后的IOC容器和它的MVC设计思想的运用。
- IOC容器  
SpringMVC在运行阶段需要用到许多的BEAN，这些bean从哪里来呢？阅读源码会知道在SpringMVC启动以先，会先通过配置文件启动并初始化好IOC容器，于是在之后的过程中只需要从IOC中获取BEAN就可以了，所以说SpringMVC是依托于强大的IOC容器的。
- 初始化组件  
在IOC容器初始化完毕后，SpringMVC会初始化9大组件，请看代码:  
```java
	/**
	 * Initialize the strategy objects that this servlet uses.
	 * <p>May be overridden in subclasses in order to initialize further strategy objects.
	 */
	protected void initStrategies(ApplicationContext context/** IOC容器 */) {

		initMultipartResolver(context);

		initLocaleResolver(context);

		initThemeResolver(context);

		/**
		 * 4、初始化HandlerMapping
         */
		initHandlerMappings(context);

		/**
		 * 5、初始化HandlerAdapters
         */
		initHandlerAdapters(context);

		initHandlerExceptionResolvers(context);

		initRequestToViewNameTranslator(context);

		initViewResolvers(context);

		initFlashMapManager(context);
	}
```
其中第4个和第5个组件是开发人员平时会接触到的，所以在这里只说一下这2个组件。
- HandlerMapping  
在SpringMVC启动的时候，通过反射机制获取所有URL和BEAN以及方法的映射关系，这个关系被封装在HandlerMapping中，请看下面的代码:
```java
@RestController
public class UserController{
    @RequestMapping("/get/user")
    public User getUser(HttpServletRequest request,HttpServletResponse response){
        //code here
    }
}
```

这段代码的核心含义是，将一个URL(/get/user)和一个BEAN(UserController的实例)的某个方法(getUser方法)建立关系，保存在HandllerMapping中，这样就可以为之后的根据URL获取对应的实现方法打下基础。

- HandlerAdapter  
HandlerMapping的建立是在SpringMVC启动的时候完成的，而HandlerAdapter的建立是在请求到达服务器的时候完成的。  
HandlerAdapter的作用是封装了用户的请求信息，并最终通过反射机制调用相应的方法。比如用户的请求是/get/user，那么HandlerAdapteper会封装以下一些用户请求信息:  
1、HttpServletRequest  
2、HttpServletResponse  
3、Header中的信息  
4、如果在URL中有参数，也会封装  
5、POST中的参数信息，包括MultiPart参数信息  
最后通过HandlerMapping找到应该调用的方法(也就是UserController的实例以及getUser方法)，通过反射传递参数并调用之。

最后要说明的是调用完方法之后，SpringMVC可以通过不同的渲染模版输出，比如Freemarker，JSP等，但因为当前RESTFULL的盛行，所以渲染部分就不详细说了。在RESTFULL中，剩余部分主要的工作就是将返回值转成JSON并输出到客户端的流中。


