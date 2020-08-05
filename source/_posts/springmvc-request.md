---
title: springMVC 请求处理流程
date: 2017-07-10
tags: 
	- spring
	- java
categories:
    - spring
keywords: spring MVC
description: 一个客户端请求是怎么样被springMVC处理的？我们来通过几个很low的简图和高大上的源码简要分析一下
---


## 首先看一个很low的简图 ##

![SpringMVC处理请求的流程](https://github.com/gamesdoa/img0/raw/master/spring/springmvc-request.jpg)


## 描述一下这个流程 ##

-  用户向服务器发送请求，请求被Spring前端控制Servelt也就是DispatcherServlet捕获；
- DispatcherServlet首先调用doService方法，从源码看首先判断是不是include请求，如果是则对request的Attribute做个快照备份， 以便doDispatch处理完之后（如果不是异步调用且未完成）进行还原，在做完快照后还对request设置一些属性，然后调用doDispatch方法。
- 根据对请求的URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；
- 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）
-  提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。 
-  Handler执行完成后，返回一个ModelAndView对象；
-  调用processDispatchResult方法处理返回的ModelAndView对象
	> 选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回一个View；
	> ViewResolver 结合Model和View，来渲染视图,然后返回到DispatcherServlet

- 将渲染结果返回给客户端。

## 概念解析 ##
- HandlerMapping：是用来查找Handler的，在SpringMVC中会处理很多请求，每个请求都需要一个Handler来处理，具体接收到一个请求后使用哪个Handler来处理呢？这就是HandlerMapping要做的事情。
- Handler：也就是处理器，它直接对应着MVC中的C也就是Controller层，它的具体表现形式有很多，可以是类，也可以是方法，如果你能想到别的表现形式也可以使用，它的类型是Object。在工程中使用@RequestMapping的所有方法都可以看成一个Handler。也就是说只要可以实际处理请求就可以是Handler。
- HandlerAdapter：是一个适配器。因为SpringMVC中的Handler可以是任意的形式，只要能处理请求就OK，但是Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法（如doService方法）。怎么让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情。

	> 通俗点的解释
	> 
	> Handler是用来干活的工具
	> 
	> HandlerMapping用于根据需要干的活找到相应的工具
	> 
	> HandlerAdapter是使用工具干活的人。

- View是用来展示数据的，
- ViewResolver用来查找View。
	> 通俗点的解释
	> 
	> 干完活后需要写报告，写报告又需要模板
	> 
	> View就是所需要的模板，内容就是Model里边的数据。
	> 
	> ViewResolver就是用来选择使用哪个模板的。

**概念解析完成之后换一种解释doDispatch方法就是：使用HandlerMapping找到干活的Handler，找到使用Handler的HandlerAdapter，让HandlerAdapter使用Handler干活，干完活后将结果写个报告通过View展示给用户。**


## 源码解析 ##
### doService方法源码 ###
	// org.springframework.web.servlet.DispatcherServlet
	@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		if (logger.isDebugEnabled()) { //日志级别为debug，输出日志
			String requestUri = urlPathHelper.getRequestUri(request);
			String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
			logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
					" processing " + request.getMethod() + " request for [" + requestUri + "]");
		}

		// 如果是include请求, 保存请求属性的快照
		// 用于该include请求完成之后恢复原始属性.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			logger.debug("Taking snapshot of request attributes before include");
			attributesSnapshot = new HashMap<String, Object>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// 处理一些request的属性，使框架对象可用于处理程序和查看对象。
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
		if (inputFlashMap != null) {
			request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
		}
		request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
		request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

		try {
			doDispatch(request, response); // 调用doDispatch方法处理流程
		}
		finally {
			if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				return;
			}
			// 如果是include请求并且请求被正常处理，恢复原始属性快照。
			if (attributesSnapshot != null) {
				restoreAttributesAfterInclude(request, attributesSnapshot);
			}
		}
	}

doService方法中首先判断是不是include请求，如果是则对request的Attribute做个快照备份， 以便doDispatch处理完之后（如果不是异步调用且未完成）进行还原，在做完快照后还对request设置了一些属性，然后调用doDispatch方法。

### doDispatch源码 ###
	// org.springframework.web.servlet.DispatcherServlet
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				// 检查是不是上传请求
				processedRequest = checkMultipart(request);
				multipartRequestParsed = processedRequest != request;

				// 根据 request 找到需要处理的handler，返回的是一个HandlerExecutionChain对象
				mappedHandler = getHandler(processedRequest, false);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// 根据 Handler 找到 HandlerAdapter
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// 处理 GET、HEAD 请求的Last-Modified
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						String requestUri = urlPathHelper.getRequestUri(request);
						logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
				// 执行相应 Interceptor的preHandle
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				try {
					// HandlerAdapter 使用 Handler处理请求
					mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
				}
				finally {
					// 如果需要异步处理，直接返回
					if (asyncManager.isConcurrentHandlingStarted()) {
						return;
					}
				}

				//当view为空时（比如，Handler返回值为void），根据request设置默认view
				applyDefaultViewName(request, mv);
				//执行相应Interceptor的postHandle
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			// 处理返回结果。包括处理异常、渲染页面、发出完成通知触发Interceptor的afterCompletion
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Error err) {
			triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
		}
		finally {
			// 判断是否执行异步请求
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				return;
			}
			// 清除上传请求的资源
			if (multipartRequestParsed) {
				cleanupMultipart(processedRequest);
			}
		}
	}

### doDispatch方法解析 ###
doDispatch大体可以分为两部分：处理请求和渲染页面。

#### 处理请求 ####
> 开头部分先定义了几个变量，在后面要用到，如下：

> - HttpServletRequestprocessedRequest：实际处理时所用的request，如果不是上传请求则直接使用接收到的request，否则封装为上传类型的request。
- HandlerExecutionChainmappedHandler：处理请求的处理器链（包含处理器和对应的Interceptor）。
- booleanmultipartRequestParsed：是不是上传请求的标志。
- ModelAndViewmv：封装Model和View的容器。
- ExceptiondispatchException：处理请求过程中抛出的异常。需要注意的是它并不包含渲染过程抛出的异常。

> doDispatch中首先检查是不是上传请求，如果是上传请求，则将request转换为Multi-partHttpServletRequest，并将multipartRequestParsed标志设置为true。其中使用到了Multipart-Resolver。
> 
> 然后通过getHandler方法获取Handler处理器链，其中使用到了HandlerMapping，返回值为HandlerExecutionChain类型，其中包含着与当前request相匹配的Interceptor和Handler。

getHandler代码如下：

	// org.springframework.web.servlet.DispatcherServlet
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		for (HandlerMapping hm : this.handlerMappings) {
			if (logger.isTraceEnabled()) {
				logger.trace(
						"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
			}
			HandlerExecutionChain handler = hm.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
		return null;
	}

> 其中hm.getHandler()方法如下：

	//org.springframework.web.servlet.handler.AbstractHandlerMapping
	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		Object handler = getHandlerInternal(request);
		if (handler == null) {
			handler = getDefaultHandler();
		}
		if (handler == null) {
			return null;
		}
		// Bean name or resolved handler?
		if (handler instanceof String) {
			String handlerName = (String) handler;
			handler = getApplicationContext().getBean(handlerName);
		}
		return getHandlerExecutionChain(handler, request);
	}

> 执行时先依次执行Interceptor的preHandle方法，最后执行Handler，返回的时候按相反的顺序执行Interceptor的postHandle方法。就好像要去一个地方，Interceptor是要经过的收费站，Handler是目的地，去的时候和返回的时候都要经过加油站，但两次所经过的顺序是相反的。

获取到Handler之后接下来是处理GET、HEAD请求的Last-Modified。当浏览器第一次跟服务器请求资源（GET、Head请求）时，服务器在返回的请求头里面会包含一个Last-Modified的属性，代表本资源最后是什么时候修改的。在浏览器以后发送请求时会同时发送之前接收到的Last-Modified，服务器接收到带Last-Modified的请求后会用其值和自己实际资源的最后修改时间做对比，如果资源过期了则返回新的资源（同时返回新的Last-Modified），否则直接返回304状态码表示资源未过期，浏览器直接使用之前缓存的结果。

接下来依次调用相应Interceptor的preHandle。

处理完Interceptor的preHandle后就到了此方法最关键的地方——让HandlerAdapter使用Handler处理请求，Controller就是在这个地方执行的。这里主要使用了HandlerAdapter，具体内容在后面详细讲解。

Handler处理完请求后，如果需要异步处理，则直接返回，如果不需要异步处理，当view为空时（如Handler返回值为void），设置默认view，然后执行相应Interceptor的postHandle。设置默认view的过程中使用到了ViewNameTranslator。

#### 渲染页面 ####

接下来使用processDispatchResult方法处理前面返回的结果，其中包括处理异常、渲染页面、触发Interceptor的afterCompletion方法三部分内容。

doDispatch的异常处理结构:
> 
> doDispatch有两层异常捕获:

>> 内层是捕获在对请求进行处理的过程中抛出的异常，
> 
>> 外层主要是在处理渲染页面时抛出的。
> 
>> 内层的异常，也就是执行请求处理时的异常会设置到dispatchException变量，然后在processDispatchResult方法中进行处理，
> 
>> 外层则是处理processDispatchResult方法抛出的异常。

processDispatchResult代码如下：

	// org.springframework.web.servlet.DispatcherServlet
	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

		boolean errorView = false;
		
		//如果请求处理的过程中有异常抛出则处理异常
		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// 处理程序是否返回一个视图用于呈现？
		if (mv != null && !mv.wasCleared()) {
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
						"': assuming HandlerAdapter completed request handling");
			}
		}

		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// 如果启动了异步处理则返回
			return;
		}
		
		//发出请求处理完成的通知，触发Interceptor的afterCompletion
		if (mappedHandler != null) {
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}


- processDispatchResult方法解析：
	> 
	> processDispatchResult处理异常的方式其实就是将相应的错误页面设置到View，在其中的processHandlerException方法中用到了HandlerExceptionResolver。
	> 
	> 渲染页面具体在render方法中执行，render中首先对response设置了Local，过程中使用到了LocaleResolver，然后判断View如果是String类型则调用resolveViewName方法使用ViewResolver得到实际的View，最后调用View的render方法对页面进行具体渲染，渲染的过程中使用到了ThemeResolver。
	> 
	> 通过mappedHandler的triggerAfterCompletion方法触发Interceptor的afterCompletion方法，这里的Interceptor也是按反方向执行的。


- 最终再返回doDispatch方法
	> 在最后的finally中判断是否请求启动了异步处理，如果启动了则调用相应异步处理的拦截器，否则如果是上传请求则删除上传请求过程中产生的临时资源。

### doDispatcher的流程图 ###
![doDispatch流程图](https://github.com/gamesdoa/img0/raw/master/spring/dispatcherServlet-doDispatch.jpg)


- 左边是Interceptor相关处理方法的调用位置.
- 右边是doDispatcher方法处理过程中所涉及的组件。
- **中间是doDispatcher的处理流程图.**
> 上半部分的处理请求对应着MVC中的Controller也就是C层，
> 
> 下半部分的processDispatchResult主要对应了MVC中的View也就是V层，
> 
> M层也就是Model贯穿于整个过程中。

