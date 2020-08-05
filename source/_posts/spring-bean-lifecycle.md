---
title: springBean生命周期
date: 2017-08-15
tags: 
	- spring
	- bean生命周期
categories:
    - spring
keywords: spring bean 生命周期
description: spring经过IOC把XML定义转换成BeanDefinition之后，要经过哪些步骤才能把BeanDefinition转换成Bean的实例的呢？这个实例的生命周期又是如何的呢？
---


# Bean生命周期流程图 #
先看一个本博主理解的生命周期的流程图：
![springbean生命周期](https://github.com/gamesdoa/img0/raw/master/spring/spring-bean-lifecycle.jpg)

# 文字描述Bean生命周期 #
## Spring容器启动 ##

- 首先会使用IOC的一套机制把定义的Bean信息加载并转化成BeanDefinition存储（如果你不了解这个过程的话可以看这里[http://gamesdoa.com/spring-ioc.html](http://gamesdoa.com/spring-ioc.html "spring IOC 解析"))
- 当存储为BeanDefinition之后如果存在**BeanFactoryPostProcessor**的实现类的话就会先实例化该实现，并执行该实例的*postProcessBeanFactory*方法，该方法生命是这样的：

		public void postProcessBeanFactory(ConfigurableListableBeanFactory arg0) throws BeansException
	> 该方法可以使用arg0.getBeanDefinition("beanName") 获取beanName对应的BeanDefinition信息，既然获取到了BeanDefinition那么就可以做很多操作了，比如修改scope，修改懒加载，等等，具体可以做哪些操作可以参看**org.springframework.beans.factory.config.BeanDefinition**，当然也可以直接修改属性值，使用如下代码：
	> 
	> *beanDefinition.getPropertyValues().addPropertyValue("propertyName", "propertyValue");* 
	> 
	> 这里相当于直接修改XML文件内的设值。针对BeanDefinition属性信息的更多操作参看**org.springframework.beans.MutablePropertyValues**
	
- 处理完BeanDefinition之后，如果存在**BeanPostProcessor**的实现的话会接着实例化该接口

- 接下来如果存在**InstantiationAwareBeanPostProcessorAdapter**的实现的话会接着实例化该接口

- 判断该 BeanDefinition 的scope属性
	-	如果scope属性是prototype，那么直接略过，将初始化的时间交给业务程序，只有在业务程序调用该bean的时候才会再进行初始化过程
	
	-	如果scope属性是singleton，并且不是懒加载的话，将立刻开始初始化过程
	
## Bean初始化过程 ##

- 首先执行 InstantiationAwareBeanPostProcessorAdapter 的 postProcessBeforeInstantiation 方法，这个执行将会在实例化bean之前调用，如果有需要的话可以在这里处理一些信息，比如安全验证什么的，该方法的声明如下：
	
			// beanClass 需要实例化的Bean的Class
			// beanName 配置的该Bean的id/name
			public Object postProcessBeforeInstantiation(Class beanClass,
                                                 String beanName) throws BeansException
- 接下来就调用bean的构造器
- 然后执行 InstantiationAwareBeanPostProcessorAdapter 的 postProcessPropertyValues 方法，这个执行就是将 BeanDefinition 中的属性值设置给生成的实例对应的属性。
- 如果bean实现了 BeanNameAware 的话就执行该接口的 setBeanName 方法
- 如果bean实现了 BeanFactoryAware 的话就执行该接口的 setBeanFactory 方法
- 如果bean实现了 ApplicationContextAware 的话就执行该接口的 setApplicationContext 方法
	>上面这三个实现的执行顺序是固定的
- 然后执行 **BeanPostProcessor** 的 postProcessBeforeInitialization 方法，这时候bean已经生成了，可以根据需求在这里对对象进行修改，该方法的声明是这样的
	
			//arg0 生成出来的对象
			//arg1 生成出来的对象对应的Id/name
			public Object postProcessBeforeInitialization(Object arg0, String arg1)
            		throws BeansException
	> **BeanPostProcessor 针对容器中的所有Bean定制初始化的过程，该接口中包含两个方法，postProcessBeforeInitialization和postProcessAfterInitialization。**
	> 
	> **postProcessBeforeInitialization 方法在容器中的Bean初始化之前执行** 
	> 
	> **postProcessAfterInitialization 方法在容器中的Bean初始化之后执行。**

- 如果bean实现了 InitializingBean 的话就执行该接口的 afterPropertiesSet 方法，该方法可以在Bean属性值设置好之后做一些操作，比如检查Bean中某个属性是否被正常的设置好值了。(存在一个问题如果实现该接口就跟spring进行了强耦合，因此spring建议使用下一个步骤这种方式来处理就是在XML定义init-method 属性的方式)
- 在xml声明bean的时候如果使用了 init-method 属性则调用<bean>的 init-method 属性指定的初始化方法
- 然后执行 BeanPostProcessor 的 postProcessAfterInitialization 方法，可以根据需求在这里对对象进行修改，该方法的声明是这样的
	
			//arg0 生成出来的对象
			//arg1 生成出来的对象对应的Id/name
			public Object postProcessAfterInitialization(Object arg0, String arg1)
            		throws BeansException

- 首先执行 InstantiationAwareBeanPostProcessorAdapter 的 postProcessAfterInitialization 方法，这个执行将会在实例化Bean之后调用，如果有需要的话可以在这里处理一些信息，比如安全验证什么的，该方法的声明如下：
	
			public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException

## Bean销毁过程 ##

- 如果scope属性是prototype，则在GC时销毁该实例，
- 如果scope属性是singleton，则需要在容器销毁时才销毁该实例

调回过程如下： 

- 如果Bean实现了 DisposableBean 接口，则调用该接口的 destroy 方法，可以在销毁Bean之前做一些操作，比如释放资源啥的。(存在一个问题如果实现该接口就跟spring进行了强耦合，因此spring建议使用下一个步骤这种方式来处理就是在XML定义 destroy-method 属性的方式)
- 在xml声明bean的时候如果使用了 destroy-method 属性则调用<bean>的 destroy-method 属性指定的销毁方法


至此：Bean的整个生命周期的描述完毕。

> 备注：实现*Aware接口 可以在Bean中使用Spring框架的一些对象，例如在我们这里使用过的BeanNameAware，BeanFactoryAware，ApplicationContextAware等。
> 
> ApplicationContextAware: 获得ApplicationContext对象,可以用来获取所有Bean definition的名字。
> 
> BeanFactoryAware:获得BeanFactory对象，可以用来检测Bean的作用域。
> 
> BeanNameAware:获得Bean在配置文件中定义的名字。
> 
> ResourceLoaderAware:获得ResourceLoader对象，可以获得classpath中某个文件。
> 
> ServletContextAware:在一个MVC应用中可以获取ServletContext对象，可以读取context中的参数。
> 
> ServletConfigAware在一个MVC应用中可以获取ServletConfig对象，可以读取config中的参数。
 
# 代码演示Bean生命周期 #

在代码展示的时候我们先声明一个简单的SpringBean ， 为了演示需要，我们让这个Bean尽可能的实现所有初始化以及销毁时需要经过的接口类，在Bean声明之后我们顺着bean生命周期的流程图上的路线，把经过的各个接口都实现一下，以便我们查看效果。

## Spring Bean ##

首先来声明一个简单的 Spring bean，代码如下：

	package com.gamesdoa.test.spring.lifecycle;

	import org.springframework.beans.BeansException;
	import org.springframework.beans.factory.*;
	import org.springframework.context.ApplicationContext;
	import org.springframework.context.ApplicationContextAware;
	
	/**
	 * Created by gamesdoa
	 */
	public class Product implements BeanFactoryAware, BeanNameAware,
	        ApplicationContextAware, InitializingBean, DisposableBean {
	
	    private String name;
	    private String description;
	    private int stock;
	
	    private BeanFactory beanFactory;
	    private String beanName;
	    private ApplicationContext applicationContext;
	
	    public Product() {
	        System.out.println("4、  【构造器】调用Product的构造器");
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        System.out.println("4、  【注入属性】注入属性name");
	        this.name = name;
	    }
	
	    public int getStock() {
	        return stock;
	    }
	
	    public void setStock(int stock) {
	        System.out.println("4、  【注入属性】注入属性stock");
	        this.stock = stock;
	    }
	
	    public String getDescription() {
	        return description;
	    }
	
	    public void setDescription(String description) {
	        System.out.println("4、  【注入属性】注入属性description");
	        this.description = description;
	    }
	
	    @Override
	    public String toString() {
	        return "4、  Product [name=" + name + ", description="
	                + description + ", stock=" + stock + "]";
	    }
	
	    // 覆盖实现BeanFactoryAware接口方法
	    @Override
	    public void setBeanFactory(BeanFactory arg0) throws BeansException {
	        System.out.println("4、  【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
	        this.beanFactory = arg0;
	    }
	
	    // 覆盖实现BeanNameAware接口方法
	    @Override
	    public void setBeanName(String arg0) {
	        System.out.println("4、  【BeanNameAware接口】调用BeanNameAware.setBeanName()");
	        this.beanName = arg0;
	    }
	
	    // 覆盖实现ApplicationContextAware接口方法
	    @Override
	    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
	        System.out.println("4、  【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()");
	        this.applicationContext = applicationContext;
	    }
	
	    // 覆盖实现InitializingBean接口方法
	    @Override
	    public void afterPropertiesSet() throws Exception {
	        System.out.println("4、  【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
	    }
	
	    // 覆盖实现DiposibleBean接口方法
	    @Override
	    public void destroy() throws Exception {
	        System.out.println("4、  【DiposibleBean接口】调用DiposibleBean.destory()");
	    }
	
	    // 通过XML配置的Bean的init-method属性指定的初始化方法
	    public void myInit() {
	        System.out.println("4、  【init-method】调用<bean>的init-method属性指定的初始化方法");
	    }
	
	    // 通过XML配置的Bean的destroy-method属性指定的销毁方法
	    public void myDestory() {
	        System.out.println("4、  【destroy-method】调用<bean>的destroy-method属性指定的销毁方法");
	    }
	}


## BeanFactoryPostProcessor ##
接下来我们看一下经过的第一个接口：BeanFactoryPostProcessor ， 实现如下：
	
	package com.gamesdoa.test.spring.lifecycle;

	import org.springframework.beans.BeansException;
	import org.springframework.beans.factory.config.BeanDefinition;
	import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
	import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
	
	/**
	 * Created by gamesdoa.
	 */
	public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	
	    public MyBeanFactoryPostProcessor() {
	        super();
	        System.out.println("1、  我是BeanFactoryPostProcessor接口实现类的构造器！！");
	    }
	
	    @Override
	    public void postProcessBeanFactory(ConfigurableListableBeanFactory arg0) throws BeansException {
	        System.out.println("1、  调用 BeanFactoryPostProcessor 接口的 postProcessBeanFactory 方法");
	        BeanDefinition bd = arg0.getBeanDefinition("product");
	        bd.getPropertyValues().addPropertyValue("stock", "2000");
	    }
	}

## BeanPostProcessor ##
接下来是经过的第二个接口： BeanPostProcessor ， 实现如下：

	package com.gamesdoa.test.spring.lifecycle;

	import org.springframework.beans.BeansException;
	import org.springframework.beans.factory.config.BeanPostProcessor;
	
	/**
	 * Created by gamesdoa.
	 */
	public class MyBeanPostProcessor implements BeanPostProcessor {
	
	    public MyBeanPostProcessor() {
	        super();
	        System.out.println("2、  我是 BeanPostProcessor 接口实现类的构造器！！");
	    }
	
	    @Override
	    public Object postProcessAfterInitialization(Object arg0, String arg1) throws BeansException {
	        System.out.println("2、  调用 BeanPostProcessor 接口的 postProcessAfterInitialization 方法对属性进行更改！");
	        return arg0;
	    }
	
	    @Override
	    public Object postProcessBeforeInitialization(Object arg0, String arg1) throws BeansException {
	        System.out.println("2、  调用 BeanPostProcessor 接口的 postProcessBeforeInitialization 方法对属性进行更改！");
	        return arg0;
	    }
	}

## InstantiationAwareBeanPostProcessorAdapter ##
接下来是第三个接口： InstantiationAwareBeanPostProcessorAdapter ，实现如下：

	package com.gamesdoa.test.spring.lifecycle;
	
	import org.springframework.beans.BeansException;
	import org.springframework.beans.PropertyValues;
	import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessorAdapter;
	
	import java.beans.PropertyDescriptor;
	
	/**
	 * Created by gamesdoa.
	 */
	public class MyInstantiationAwareBeanPostProcessor extends
	        InstantiationAwareBeanPostProcessorAdapter {
	
	    public MyInstantiationAwareBeanPostProcessor() {
	        super();
	        System.out.println("3、  我是InstantiationAwareBeanPostProcessorAdapter接口实现类的构造器！！");
	    }
	
	    // 接口方法、实例化Bean之前调用
	    @Override
	    public Object postProcessBeforeInstantiation(Class beanClass,String beanName) throws BeansException {
	        System.out.println("3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessBeforeInstantiation 方法");
	        return null;
	    }
	
	    // 接口方法、实例化Bean之后调用
	    @Override
	    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
	        System.out.println("3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessAfterInitialization 方法");
	        return bean;
	    }
	
	    // 接口方法、设置属性时调用
	    @Override
	    public PropertyValues postProcessPropertyValues(PropertyValues pvs,PropertyDescriptor[] pds, 
	                                                    Object bean, String beanName) throws BeansException {
	        System.out.println("3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessPropertyValues 方法");
	        return pvs;
	    }
	}

## 定义配置文件 ##
配置文件如下spring-beans.xml：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:p="http://www.springframework.org/schema/p"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans
	            http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
	
	    <bean id="beanPostProcessor" class="com.gamesdoa.test.spring.lifecycle.MyBeanPostProcessor"/>
	
	    <bean id="instantiationAwareBeanPostProcessor"
	          class="com.gamesdoa.test.spring.lifecycle.MyInstantiationAwareBeanPostProcessor"/>
	
	    <bean id="beanFactoryPostProcessor"
	          class="com.gamesdoa.test.spring.lifecycle.MyBeanFactoryPostProcessor"/>
	
	    <bean id="product" class="com.gamesdoa.test.spring.lifecycle.Product" init-method="myInit"
	          destroy-method="myDestory" scope="singleton" p:name="iphone 100" p:description="我是未来的iphone 100"
	          p:stock="900" />
	</beans>

## 测试类 ##

	package com.gamesdoa.test.spring.lifecycle;

	import org.springframework.context.ApplicationContext;
	import org.springframework.context.support.ClassPathXmlApplicationContext;
	
	/**
	 * Created by gamesdoa.
	 */
	public class BeanLifeCycleTest {
	
	    public static void main(String[] args) {
	        System.out.println("0、  现在开始初始化容器");
	        ApplicationContext factory = new ClassPathXmlApplicationContext("spring/spring-beans.xml");
	        System.out.println("0、  容器初始化成功");
	        //得到Product，并使用
	        Product product = factory.getBean("product",Product.class);
        	System.out.println("0、  hashCode : " + product.hashCode());
	        System.out.println(product);

	        product = factory.getBean("product",Product.class);
        	System.out.println("0、  hashCode : " + product.hashCode());
	        System.out.println(product);
	
	        System.out.println("0、  现在开始关闭容器！");
	        ((ClassPathXmlApplicationContext)factory).registerShutdownHook();
	    }
	}

## 运行结果 ##

这里我们分别看一下scope 分别为 singleton 和 prototype 时的运行结果：

### singleton 的运行结果 ###

	0、  现在开始初始化容器
	1、  我是BeanFactoryPostProcessor接口实现类的构造器！！
	1、  调用 BeanFactoryPostProcessor 接口的 postProcessBeanFactory 方法
	2、  我是 BeanPostProcessor 接口实现类的构造器！！
	3、  我是InstantiationAwareBeanPostProcessorAdapter接口实现类的构造器！！

	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessBeforeInstantiation 方法
	4、  【构造器】调用Product的构造器
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessPropertyValues 方法
	4、  【注入属性】注入属性description
	4、  【注入属性】注入属性name
	4、  【注入属性】注入属性stock
	4、  【BeanNameAware接口】调用BeanNameAware.setBeanName()
	4、  【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
	4、  【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()
	2、  调用 BeanPostProcessor 接口的 postProcessBeforeInitialization 方法对属性进行更改！
	4、  【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
	4、  【init-method】调用<bean>的init-method属性指定的初始化方法
	2、  调用 BeanPostProcessor 接口的 postProcessAfterInitialization 方法对属性进行更改！
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessAfterInitialization 方法

	0、  容器初始化成功

	0、  hashCode : 1345604737
	4、  Product [name=iphone 100, description=我是未来的iphone 100, stock=2000]
	0、  hashCode : 1345604737
	4、  Product [name=iphone 100, description=我是未来的iphone 100, stock=2000]

	0、  现在开始关闭容器！
	4、  【DiposibleBean接口】调用DiposibleBean.destory()
	4、  【destroy-method】调用<bean>的destroy-method属性指定的销毁方法


### prototype的运行结果 ###

	0、  现在开始初始化容器
	1、  我是BeanFactoryPostProcessor接口实现类的构造器！！
	1、  调用 BeanFactoryPostProcessor 接口的 postProcessBeanFactory 方法
	2、  我是 BeanPostProcessor 接口实现类的构造器！！
	3、  我是InstantiationAwareBeanPostProcessorAdapter接口实现类的构造器！！
	0、  容器初始化成功
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessBeforeInstantiation 方法

	4、  【构造器】调用Product的构造器
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessPropertyValues 方法
	4、  【注入属性】注入属性description
	4、  【注入属性】注入属性name
	4、  【注入属性】注入属性stock
	4、  【BeanNameAware接口】调用BeanNameAware.setBeanName()
	4、  【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
	4、  【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()
	2、  调用 BeanPostProcessor 接口的 postProcessBeforeInitialization 方法对属性进行更改！
	4、  【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
	4、  【init-method】调用<bean>的init-method属性指定的初始化方法
	2、  调用 BeanPostProcessor 接口的 postProcessAfterInitialization 方法对属性进行更改！
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessAfterInitialization 方法

	0、  hashCode : 1431641429
	4、  Product [name=iphone 100, description=我是未来的iphone 100, stock=2000]

	4、  【构造器】调用Product的构造器
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessPropertyValues 方法
	4、  【注入属性】注入属性description
	4、  【注入属性】注入属性name
	4、  【注入属性】注入属性stock
	4、  【BeanNameAware接口】调用BeanNameAware.setBeanName()
	4、  【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
	4、  【ApplicationContextAware接口】调用ApplicationContextAware.setApplicationContext()
	2、  调用 BeanPostProcessor 接口的 postProcessBeforeInitialization 方法对属性进行更改！
	4、  【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
	4、  【init-method】调用<bean>的init-method属性指定的初始化方法
	2、  调用 BeanPostProcessor 接口的 postProcessAfterInitialization 方法对属性进行更改！
	3、  调用 InstantiationAwareBeanPostProcessor 接口的 postProcessAfterInitialization 方法

	0、  hashCode : 1190716215
	4、  Product [name=iphone 100, description=我是未来的iphone 100, stock=2000]

	0、  现在开始关闭容器！

### 总结 ###

为了能够直观的看出 scope 分别为 singleton 和 prototype 的区别，我这里专门使用了两次该Bean，我们可以看出

- 当 scope 配置为 singleton 的时候，该bean的初始化是在容器启动时就实例化好了的，在业务程序中使用的时候每次获取的对象也都是同一个，你可以看到两次获取的 hashCode 是一致的。
- 当 scope 配置为 prototype 的时候，该bean的初始化是在业务程序使用该对象的时候即时初始化的，每次调用该Bean的时候都会重新初始化该对象，你可以看到两次获取的 hashCode 是不一致的。

# 结束语 #

至此，spring bean的生命周期我们就解析完了，该篇中我们先给出了流程图，然后又采用文字进行了描述，最后使用代码给出了生命周期的演示。