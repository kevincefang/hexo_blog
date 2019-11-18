---
title: SpringMVC源码分析-DispatcherServlet
date: 2018-08-28
tags: 
- spring
- 源码分析
categories: spring
---

>`SpringMvc`是Spring框架的`web`模块,是目前最常用的web框架之一.下面让我们一起通过阅读源码来揭秘一下他的神奇之处吧.

#### SpringMvc初始化

`SpringMVC`框架是依托于`Spring`容器。`Spring`初始化的过程其实就是`IoC`容器启动的过程，也就是上下文建立的过程。

#### ServletContext

每一个web应用中都有一个Servlet上下文。`servlet`容器提供一个全局上下文的环境，这个上下文环境将成为其他`IoC`容器的宿主环境，例如：`WebApplicationContext`就是作为`ServletContext`的一个属性存在。

#### WebApplicationContext

在使用`SpringMVC`的时候，通常需要在`web.xml`文件中配置：

```xml
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```

`ContextLoaderListener` 实现了`ServletContextListener`接口，在`SpringMVC`中作为监听器的存在，当`servlet`容器启动时候，会调用`contextInitialized`进行一些初始化的工作。而`ContextLoaderListener`中`contextInitialized`的具体实现在`ContextLoader`类中的`initWebApplicationContext`方法中。

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
        throw new IllegalStateException(
            "Cannot initialize context because there is already a root application context present - " +
            "check whether you have multiple ContextLoader* definitions in your web.xml!");
    }

    Log logger = LogFactory.getLog(ContextLoader.class);
    servletContext.log("Initializing Spring root WebApplicationContext");
    if (logger.isInfoEnabled()) {
        logger.info("Root WebApplicationContext: initialization started");
    }
    long startTime = System.currentTimeMillis();

    try {
        // Store context in local instance variable, to guarantee that
        // it is available on ServletContext shutdown.
        if (this.context == null) {
            this.context = createWebApplicationContext(servletContext);
        }
        if (this.context instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent ->
                    // determine parent for root web application context, if any.
                    ApplicationContext parent = loadParentContext(servletContext);
                    cwac.setParent(parent);
                }
                configureAndRefreshWebApplicationContext(cwac, servletContext);
            }
        }
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

        ClassLoader ccl = Thread.currentThread().getContextClassLoader();
        if (ccl == ContextLoader.class.getClassLoader()) {
            currentContext = this.context;
        }
        else if (ccl != null) {
            currentContextPerThread.put(ccl, this.context);
        }

        if (logger.isDebugEnabled()) {
            logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
                         WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
        }
        if (logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
        }

        return this.context;
    }
    catch (RuntimeException ex) {
        logger.error("Context initialization failed", ex);
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
        throw ex;
    }
    catch (Error err) {
        logger.error("Context initialization failed", err);
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, err);
        throw err;
    }
}
```

上面的部分代码可以看出，初始化时候通过`createWebApplicationContext(servletContext);`声明一个`WebApplicationContext`并赋值给`ServletContext`的`org.springframework.web.context.WebApplicationContext.ROOT`属性，作为`WebApplicationContext`的根上下文（root context）。

#### DispatcherServlet

在加载完`<context-param>`和`<listener>`之后，容器将加载配置了`load-on-startup`的`servlet`。

```
<servlet>
    <servlet-name>example</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>example</servlet-name>
    <url-pattern>/example/*</url-pattern>
</servlet-mapping>
```

`DispatcherServlet`在初始化的过程中，会建立一个自己的`IoC`容器上下文`Servlet WebApplicationContext`，会以`ContextLoaderListener`建立的根上下文作为自己的父级上下文。`DispatcherServlet`持有的上下文默认的实现类是`XmlWebApplicationContext`。`Servlet`有自己独有的`Bean`空间，也可以共享父级上下文的共享`Bean`，当然也存在配置有含有一个`root WebApplicationContext`配置。

`DispatcherServlet`最为SpringMVC核心类，起到了前端控制器（Front controller）的作用，负责请求分发等工作。

![1](http://p066mj5r9.bkt.clouddn.com/static/images/20180828/1.png)

从类图中可以看出，`DispatcherServlet`的继承关系大致如此：

```
DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet -> GenericServlet
```

从继承关系上可以得出结论，`DispatcherServlet`本质上还是一个`Servlet`。`Servlet`的生命周期大致分为三个阶段：

- 初始化阶段 init方法
- 处理请求阶段 service方法
- 结束阶段 destroy方法

这里就重点关注`DispatcherServlet`在这三个阶段具体做了那些工作。 

#### DispatcherServlet初始化

`DispatcherServlet`的`init()`的实现在其父类`HttpServletBean`中。 

```java
public final void init() throws ServletException {
    if (logger.isDebugEnabled()) {
        logger.debug("Initializing servlet '" + getServletName() + "'");
    }

    // Set bean properties from init parameters.
    try {
        PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
        ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
        bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
        initBeanWrapper(bw);
        bw.setPropertyValues(pvs, true);
    }
    catch (BeansException ex) {
        if (logger.isErrorEnabled()) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
        }
        throw ex;
    }

    // Let subclasses do whatever initialization they like.
    initServletBean();

    if (logger.isDebugEnabled()) {
        logger.debug("Servlet '" + getServletName() + "' configured successfully");
    }
}
```

以上部分源码描述的过程是通过读取`<init-param>`的配置元素，读取到`DispatcherServlet`中，配置相关`bean`的配置。完成配置后调用`initServletBean`方法来创建`Servlet WebApplicationContext`。

`initServletBean`方法在`FrameworkServlet`类中重写了： 

```java
protected final void initServletBean() throws ServletException {
    ...

        try {
            this.webApplicationContext = initWebApplicationContext();
            initFrameworkServlet();
        }
    ...
}

protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        onRefresh(wac);
    }

    if (this.publishContext) {
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
                              "' as ServletContext attribute with name [" + attrName + "]");
        }
    }

    return wac;
}
```

上文提到`Servlet`容器在启动的时候，通过`ContextLoaderListener`创建一个根上下文，并配置到`ServletContext`中。可以看出`FrameworkServlet`这个类做的作用是用来创建`WebApplicationContext`上下文的。大致过程如下：

* 首先检查`webApplicationContext`是否通过构造函数注入，如果有的话，直接使用，并将根上下文设置为父上下文。

* 如果`webApplicationContext`没有注入，则检查是否在`ServletContext`已经注册过，如果已经注册过，直接返回使用。

* 如果没有注册过，将重新新建一个`webApplicationContext`。将根上下文设置为父级上下文。

* 不管是何种策略获取的`webApplicationContext`，都将会调用`onRefresh`方法，`onRefresh`方法会调用`initStrategies`方法，通过上下文初始化`HandlerMappings`、`HandlerAdapters`、`ViewResolvers`等等。

* 最后，同样会将所得`webApplicationContext`注册到`ServletContext`中。

而`initFrameworkServlet()`默认的实现是空的。这也可算是`SpingMVC`留的一个扩展点。

#### DispatcherServlet处理请求

纵观`SpringMVC`的源码，大量运用模板方法的设计模式。`Servlet`的`service`方法也不例外。`FrameworkServlet`类重写`service`方法：

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
    if (HttpMethod.PATCH == httpMethod || httpMethod == null) {
        processRequest(request, response);
    }
    else {
        super.service(request, response);
    }
}
```

如果请求的方法是`PATCH`或者空，直接调用`processRequest`方法（后面会详细解释）；否则，将调用父类的`service`的方法，即`HttpServlet`的`service`方法, 而这里会根据请求方法，去调用相应的`doGet`、`doPost`、`doPut`......

而`doXXX`系列方法的实现并不是`HttpServlet`类中，而是在`FrameworkServlet`类中。在`FrameworkServlet`中`doXXX`系列实现中，都调用了上面提到的`processRequest`方法：

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }

        if (logger.isDebugEnabled()) {
            if (failureCause != null) {
                this.logger.debug("Could not complete request", failureCause);
            }
            else {
                if (asyncManager.isConcurrentHandlingStarted()) {
                    logger.debug("Leaving response open for concurrent processing");
                }
                else {
                    this.logger.debug("Successfully completed request");
                }
            }
        }

        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

为了避免子类重写它，该方法用`final`修饰。

* 首先调用`initContextHolders`方法,将获取到的`localeContext`、`requestAttributes`、`request`绑定到线程上。

* 然后调用`doService`方法，`doService`具体是由`DispatcherServlet`类实现的。

*  `doService`执行完成后，调用`resetContextHolders`，解除`localeContext`等信息与线程的绑定。

* 最终调用`publishRequestHandledEvent`发布一个处理完成的事件。

`DispatcherServlet`类中的`doService`方法实现会调用`doDispatch`方法，这里请求分发处理的主要执行逻辑。

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = (processedRequest != request);

                // Determine handler for the current request.
                mappedHandler = getHandler(processedRequest);
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }

                // Determine handler adapter for the current request.
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (logger.isDebugEnabled()) {
                        logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                    }
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

                // Actually invoke the handler.
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }

                applyDefaultViewName(processedRequest, mv);
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
            catch (Throwable err) {
                // As of 4.3, we're processing Errors thrown from handler methods as well,
                // making them available for @ExceptionHandler methods and other scenarios.
                dispatchException = new NestedServletException("Handler dispatch failed", err);
            }
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Throwable err) {
            triggerAfterCompletion(processedRequest, response, mappedHandler,
                    new NestedServletException("Handler processing failed", err));
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            }
            else {
                // Clean up any resources used by a multipart request.
                if (multipartRequestParsed) {
                    cleanupMultipart(processedRequest);
                }
            }
        }
    }
```

`doDispatch`主要流程是：

* 先判断是否`Multipart`类型的请求。如果是则通过`multipartResolver`解析`request` 

* 通过`getHandler`方法找到从`HandlerMapping`找到该请求对应的`handler`,如果没有找到对应的`handler`则抛出异常。

* 通过`getHandlerAdapter`方法找到`handler`对应的`HandlerAdapter` 

* 如果有拦截器，执行拦截器`preHandler`方法

* `HandlerAdapter`执行`handle`方法处理请求，返回`ModelAndView`。

* 如果有拦截器，执行拦截器`postHandle`方法

* 然后调用`processDispatchResult`方法处理请求结果，封装到`response`中。

#### SpingMVC 请求处理流程

`SpringMVC`框架是围绕`DispatcherServlet`设计的。`DispatcherServlet`负责将请求分发给对应的处理程序。从网上找了两个图，可以大致了解`SpringMVC`的框架对请求的处理流程。

![1](http://fhaoer.com/hexo_blog/static/images/20180828/2.webp)

![1](http://fhaoer.com/hexo_blog/static/images/20180828/3.webp)

* 用户发送请求，`Front Controller`（`DispatcherServlet`）根据请求信息将请求委托给对应的`Controller`进行处理。

* `DispatcherServlet`接收到请求后，`HandlerMapping`将会把请求封装为`HandlerExecutionChain`，而`HandlerExecutionChain`包含请求的所有信息，包括拦截器、Handler处理器等。

* `DispatcherServlet`会找到对应的`HandlerAdapter`，并调用对应的处理方法，并返回一个`ModelAndView`对象。

* `DispatcherServlet`会将`ModelAndView`对象传入`View`层进行渲染。

* 最终`DispatcherServlet`将渲染好的`response`返回给用户。

#### 总结

本文主要分析`SpringMVC`中`DispatcherServlet`的初始化、请求流传过程等。

发现了`SpringMVC`中在`DispatcherServlet`的实现过程中运用了模板方法设计模式，看到`SpringMVC`中留给用户可扩展的点也有很多，体会到`Open for extension, closed for modification`的设计原则。