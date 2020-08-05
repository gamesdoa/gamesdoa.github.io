---
title: spring AOP解析
date: 2017-08-05
tags: 
	- spring
	- aop
categories:
    - spring
keywords: spring aop 面向切面编程
description: spring几大核心组件中有一个就是AOP(Aspect Oriented Programming)，那么什么是AOP呢？我们应该怎么样使用？它的原理是什么？它对我们的编程有什么影响?下面我们就来分析分析
---


# 什么是AOP #
AOP全称：Aspect Oriented Programming ，翻译成中文就是面向切面编程，主要的作用就是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术.

AOP是spring核心组件中的一个比较重要的组成部分，可以利用AOP对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

AOP一般用到哪些地方呢？
> 一般在我们开发工程中经常使用AOP的地方主要是：日志记录，性能统计，安全控制，事务处理，异常处理等。

AOP联盟对于AOP给出来一个框架，其中的概念如下：

1. join point（连接点）：是程序执行中的一个精确执行点，例如类中的一个方法。它是一个抽象的概念，在实现AOP时，并不需要去定义一个join point。
2. point cut（切入点）：本质上是一个捕获连接点的结构。在AOP中，可以定义一个point cut，来捕获相关方法的调用。
3. advice（通知）：是point cut的执行代码，是执行“方面”的具体逻辑。
4. aspect（方面）：point cut和advice结合起来就是aspect，它类似于OOP中定义的一个类，但它代表的更多是对象间横向的关系。
5. introduce（引入）：为对象引入附加的方法或属性，从而达到修改对象结构的目的。有的AOP工具又将其称为mixin。

Spring在实现AOP的时候遵循了该框架，但是也有一些变化：

- Advice接口 : 扩展了很多子接口，例如：BeforeAdvice、AfterAdvice、AfterReturningAdvice、ThrowsAdvice等，所有实现可以查看org.springframework.aop包下的声明。
- Pointcut接口 ： 切入点决定要切入哪些方法，spring采用Pointcut作为切入点的抽象，其中getMethodMatcher()方法返回MethodMatcher，是一个定义的匹配规则，比如正则表达式。
- Advisor接口 ： 通知器或者通知者，作为通知器当然是需要知道通知什么，因此：Advisor依赖于Advice，源码中也可以看到有Advice getAdvice()方法。Advisor的子接口PointcutAdvisor还依赖于Pointcut接口源码中Pointcut getPointcut()方法。也就是说：接口接口更确切的定位应该是包含了要通知谁和要通知什么，因此需要获取Advice和Pointcut。

> Spring AOP相关的接口声明都在 **org.springframework.aop** 包下，需要了解更多可以自行查看。

# Spring AOP如何使用 #
我们用过一个简单的例子来看一下Spring 中如何使用AOP：

- 使用AOP的话需要引入相应的JAR包：
	- spring-beans
	- spring-aop
	- aspectjweaver
	- cglib-nodep
	- aopalliance

- 我们采用普通的面向接口开发的形式编写我们的java代码：

接口声明：

	package com.gamesdoa.test.aop;
	public interface AopService {
	    String saveBean();
	    String delBean();
	}

实现：

	package com.gamesdoa.test.aop.impl;

	import com.gamesdoa.test.aop.AopService;

	public class AopServiceImpl implements AopService {
	    public String saveBean() {
	        System.out.println("I'm save!");
	        return "save";
	    }
	
	    public String delBean() {
	        System.out.println("I'm del!");
	        return "del";
	    }
	}
定义一个横切关注点，用于方法调用前后输出不同的时间：

	package com.gamesdoa.test.aop;

	public class AopHandler {
		//方法开始前输出进入时间
	    public void enterTime() {
	        System.out.println("enter time:" + System.currentTimeMillis());
	    }
		//方法接受后输出离开时间
	    public void leaveTime() {
	        System.out.println("leave time:" + System.currentTimeMillis());
	    }
	
	}

定义一个XML文件spring-aop.xml：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:aop="http://www.springframework.org/schema/aop"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	        http://www.springframework.org/schema/aop
	        http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">
	
	    <bean id="aopServiceImpl" class="com.gamesdoa.test.aop.impl.AopServiceImpl" />
	    <bean id="timeHandler" class="com.gamesdoa.test.aop.AopHandler" />
	
	    <aop:config proxy-target-class="true">
	        <aop:aspect id="time" ref="timeHandler">
	            <aop:pointcut id="addAllMethod" expression="execution(* com.gamesdoa.test.aop.AopService.*(..))" /><!-- 哪些方法需要织入 -->
				<!-- 这里还可以是异常时，返回前，等等详情查看springapi -->
	            <aop:before method="enterTime" pointcut-ref="addAllMethod" /><!-- 开始前 -->
	            <aop:after method="leaveTime" pointcut-ref="addAllMethod" /><!-- 结束后 -->
	        </aop:aspect>
	    </aop:config>
	</beans>

编写测试类：

	package com.gamesdoa.test.aop;

	import org.junit.Test;
	import org.springframework.context.ApplicationContext;
	import org.springframework.context.support.ClassPathXmlApplicationContext;

	public class TestAop {
	    @Test
	    public void testAop() {
	        ApplicationContext ac = new ClassPathXmlApplicationContext("spring-aop.xml");
	
	        AopService service = (AopService)ac.getBean("aopServiceImpl");
	        service.saveBean();
	        System.out.println("------------我是华丽的分界线------------");
	        service.delBean();
	    }
	}

接下来就是运行了，运行结果是这样的：

	enter time:1503629611721
	I'm save!
	leave time:1503629611722
	------------我是华丽的分界线------------
	enter time:1503629611722
	I'm del!
	leave time:1503629611722

可以看出来运行时先输入了进入时间，然后运行方法内的实现，最后返回结束之后输出了离开时间。