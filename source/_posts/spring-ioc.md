---
title: spring IOC解析
date: 2017-07-30
tags: 
	- spring
	- ioc
categories:
    - spring
keywords: spring ioc 控制反转
description: spring几大核心组件中有一个就是IOC(Inversion of Control)，那么什么是IOC呢？它的原理是什么？它对我们的编程有什么影响?我们应该怎么样使用？下面我们就来分析分析
---


# 什么是IOC #
IOC全称：Inversion of Control ，翻译成中文就是控制反转，主要的作用就是把我们原来在程序里面关联两个对象使用的new Object()方法，交给容器控制，也就是说将**对象的创建和依赖关系交给容器**

那么对象之间的依赖怎么表示呢？
> 可以用 xml ， properties 文件等语义化配置文件表示。

描述对象关系的文件存放在哪里？
> 可能是 classpath ， filesystem ，或者是 URL 网络资源， servletContext 等。


# Spring IOC体系结构 #

Spring IOC体系结构中有两个最重要的接口：

- BeanFactory：核心工厂接口，这个主要是创建Bean的
- BeanDefinition ： bean对象以及其相互关系的定义接口

## BeanFactory ##
![BeanFactory架构图](https://github.com/gamesdoa/img0/raw/master/spring/BeanFactory.jpg)
从这个架构图中可以看出BeanFactory的架构继承实现关系

BeanFactory作为最顶层的一个接口类，它定义了IOC容器的基本功能规范；

BeanFactory 有三个子类：ListableBeanFactory、HierarchicalBeanFactory 和AutowireCapableBeanFactory。从上图中我们可以发现最终的默认实现类是 DefaultListableBeanFactory，他实现了所有的接口。

spring为何要定义这么多层次的接口？其实这个设计主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制。
> ListableBeanFactory 接口表示这些 Bean 是可列表的;
> 
> HierarchicalBeanFactory 表示的是这些 Bean 是有继承关系的，也就是每个Bean 有可能有父 Bean;
> 
> AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则

	package org.springframework.beans.factory;

	import org.springframework.beans.BeansException;

	public interface BeanFactory {

		/**
		 * 用于取消引用FactoryBean实例，并将其与FactoryBean创建的bean进行区分
		 * 例如，如果获取名为myJndiObject的bean是FactoryBean，则获取＆myJndiObject将返回工厂，
		 * 而不返回工厂返回的实例。
		 */
		String FACTORY_BEAN_PREFIX = "&";
	
		/**
		 * 返回指定bean的一个可能是共享的或独立的实例。
		 * 如果在此工厂实例中找不到该bean，将查找父工厂。
		 * @param 要查找的bean名称
		 * @返回一个Bean的实例
		 * @throws NoSuchBeanDefinitionException如果没有bean定义指定名称
		 * @throws BeansException如果无法获取该bean
		 */
		Object getBean(String name) throws BeansException;
	
		/**
		 * 返回指定bean的一个可能是共享的或独立的实例。
		 * 如果在此工厂实例中找不到该bean，将查找父工厂。
		 * @param 要查找的bean名称
		 * @param requiredType键入bean必须匹配。可以是一个接口或超类，或匹配任意的null。
			 例如，如果值是Object.class，这个方法将无论什么类都成功返回实例。
		 * @返回一个Bean的实例
		 * @throws NoSuchBeanDefinitionException如果没有这样的bean定义
		 * @throws BeanNotOfRequiredTypeException如果bean不是必需的类型
		 * @throws BeansException如果无法创建bean 
		 */
		<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	
		/**
		 * 返回与给定对象类型唯一匹配的bean实例（如果有的话）。
		 * @param requiredType键入bean必须匹配;可以是一个接口或超类。不允许为null。
		 * 此方法进入ListableBeanFactory按类型查找区域，但也可以根据名称翻译成常规的副名称查找给定类型。
		 * 对于多组bean的更广泛的检索操作，使用ListableBeanFactory或BeanFactoryUtils。
		 * 返回匹配所需类型的单个bean的实例
		 * @throws NoSuchBeanDefinitionException如果没有找到一个匹配的bean
		 */
		<T> T getBean(Class<T> requiredType) throws BeansException;
	
		/**
		 *返回指定bean的一个可能是共享的或独立的实例。
		 * 允许指定显式构造函数参数/工厂方法参数，覆盖bean定义中指定的默认参数（如果有）。
		 * @param 要查找的bean名称
		 * @param args使用参数，如果使用明确的参数创建一个原型静态工厂方法。在任何其他情况下使用非空args值无效。
		 * @返回一个Bean的实例
		 * @throws NoSuchBeanDefinitionException如果没有这样的bean定义
		 * @throws BeanDefinitionStoreException如果已经给出了参数受影响的bean不是原型
		 * @throws BeansException如果无法创建bean
		 */
		Object getBean(String name, Object... args) throws BeansException;
	
		/**
		 *这个bean工厂是否包含一个bean定义或外部注册的具有给定名称的单例实例？
		 *如果给定的名称是别名，它将被翻译回相应的规范bean名称。
		 *如果这个工厂是分层的，如果在这个工厂实例中找不到该bean，则会在其他工厂中查找，直到找到或者所有工厂查询完。
		 *如果在相应的范围内找到匹配给定名称的bean定义或单例实例，该方法将返回true，无论找到的bean是具体还是抽象，懒加载或立即加载。
		  因此，请注意，此方法的真实返回值并不一定表示#getBean将能够获取相同名称的实例。
		 * @param 要查找的bean名称
		 * @返回是否存在具有给定名称的bean
		 */
		boolean containsBean(String name);
	
		/**
		 * 这个bean是不是单例的？ 也就是说，#getBean是否会始终返回相同的实例？
		 * 注意：此方法返回false并不能清楚的说明是单例。它也可能是非单例实例，只是对应的作用域不同。 使用#isPrototype操作来显式检查是否全局单例。
		 * 将别名转换回相应的规范bean名称。 如果在此工厂实例中找不到该bean，将查询父工厂。
		 * @param 要查找的bean名称
		 * @返回这个bean是否对应于单例实例
		 * @throws NoSuchBeanDefinitionException如果没有给定名称的bean
		 */
		boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	
		/**
		 * bean是不是一个原型，#getBean是不是会一直返回独立的实例？
		 * 注意：此方法返回false并不能清楚的说明是单例。它也可能是非单例实例，只是对应的作用域不同。 使用{@link #isSingleton}操作来显式检查共享的单例实例。
		 * 将别名转换回相应的规范bean名称。 如果在此工厂实例中找不到该bean，将查询父工厂。
		 * @param 要查找的bean名称
		 * @返回这个bean是否会始终提供独立的实例
		 * @throws NoSuchBeanDefinitionException如果没有给定名称的bean
		 */
		boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	
		/**
		 *检查给定名称的bean是否与指定的类型匹配。 
		 * 更具体地说，检查给定名称的{@link #getBean}调用是否返回可分配给指定目标类型的对象。
		 * 将别名转换回相应的规范bean名称。 如果在此工厂实例中找不到该bean，将查询父工厂。
		 * @param 要查找的bean名称
		 * @param targetType匹配的类型
		 * @return  true 如果bean类型匹配， false 如果不匹配或无法确定
		 * @throws NoSuchBeanDefinitionException如果没有给定名称的bean
		 */
		boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;
	
		/**
		 *确定具有给定名称的bean的类型。 更具体地说，确定{@link #getBean}将为给定名称返回的对象的类型。
		 * 对于{@link FactoryBean}，返回FactoryBean创建的对象类型，如{@link FactoryBean＃getObjectType（））所示。
		 * 将别名转换回相应的规范bean名称。 如果在此工厂实例中找不到该bean，将查询父工厂。
		 * @param命名要查询的bean的名称 返回bean的类型，如果不可确定，则返回null
		 * @throws NoSuchBeanDefinitionException如果没有给定名称的bean
		 */
		Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	
		/**
		 *返回给定bean名称的别名（如果有）。 当在{@link #getBean}调用中使用时，所有这些别名都指向同一个bean。
		 * 如果给定的名称是别名，将返回相应的原始bean名称和其他别名（如果有的话），原始bean名称是数组中的第一个元素。
		 * 如果在此工厂实例中找不到该bean，将查找父工厂。
		 * @param 要查找的bean名称
		 * @返回别名，如果没有，则返回一个空数组
		 */
		String[] getAliases(String name);

	}

从源码上看BeanFactory里只对IOC容器的基本行为作了定义，根本不关心你的bean是如何定义又是怎样加载的。正如我们只关心工厂里能得到什么产品对象，至于工厂是怎么生产这些对象的，这个基本接口时不需要关心的。

而要知道工厂是如何产生对象的，我们需要看具体的IOC容器实现，spring提供了许多IOC容器的实现。
> 比如XmlBeanFactory，ClasspathXmlApplicationContext等。
> 
> - XmlBeanFactory就是针对最基本的ioc容器的实现，这个IOC容器可以读取XML文件定义的BeanDefinition（XML文件中对bean的描述）。
> - ApplicationContext是Spring提供的一个高级的IoC容器，它除了能够提供IoC容器的基本功能外，还为用户提供了其他的附加服务。
> >从ApplicationContext接口的实现，我们看出其特点：
> > 

		org.springframework.context
		
		import org.springframework.beans.factory.HierarchicalBeanFactory;
		import org.springframework.beans.factory.ListableBeanFactory;
		import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
		import org.springframework.core.env.EnvironmentCapable;
		import org.springframework.core.io.support.ResourcePatternResolver;
	
		public interface ApplicationContext extends EnvironmentCapable, 
			ListableBeanFactory, HierarchicalBeanFactory,
			MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
			}

>>  1.  支持信息源，可以实现国际化。（实现MessageSource接口）
>>  2.  支持应用事件。(实现ApplicationEventPublisher接口)
>>  3.  访问资源。(实现ResourcePatternResolver接口)


## BeanDefinition ##
![BeanDefinition设计架构](https://github.com/gamesdoa/img0/raw/master/spring/BeanDefinition.jpg)

- 注 BeanDefinition是用来描述各种bean对象以及其相互的关系的定义继承了两个接口 
	> AttributeAccessor:使其具有了处理属性的能力
	> 
	> BeanMetadataElement : 使其具有了获取bean配置定义的元素 
	
看一下源码：

	package org.springframework.beans.factory.config;

	import org.springframework.beans.BeanMetadataElement;
	import org.springframework.beans.MutablePropertyValues;
	import org.springframework.core.AttributeAccessor;

	public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
	
		/**
		 * 单例表示 : "singleton".
		 * 扩展bean工厂可能会支持更多的范围。
		 */
		String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
	
		/**
		 * 标准原型范围的范围标识符：“原型”。
		 * 扩展bean工厂可能会支持更多的范围。
		 */
		String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;
	
	
		/**
		 * 角色提示，表明一个  BeanDefinition 是应用程序的主要部分。
		 * 通常对应于用户定义的bean。
		 */
		int ROLE_APPLICATION = 0;
	
	
		/**
		 * 返回此bean定义的父定义的名称（如果有）。
		 */
		String getParentName();
	
		/**
		 * 设置此bean定义的父定义的名称（如果有）。
		 */
		void setParentName(String parentName);
	
		/**
		 * 返回此bean定义的当前bean类名。
		 * 请注意，这不一定是在运行时使用的实际类名，以便于子类定义从其父类覆盖/继承类名。 
		 * 因此，不要认为这是在运行时确定的bean类型，而仅仅是在各个bean定义级别使用它来做解析的目的。
		 */
		String getBeanClassName();
	
		/**
		 * 覆盖此bean定义的bean类名称。
		 * 可以在bean工厂之后处理修改类名称，通常用解析变量替换原始类名。
		 */
		void setBeanClassName(String beanClassName);
	
		/**
		 * 返回工厂bean名称，如果有的话。
		 */
		String getFactoryBeanName();
	
		/**
		 * 指定要使用的工厂bean（如果有）。
		 */
		void setFactoryBeanName(String factoryBeanName);
	
		/**
		 * 返回工厂方式，如果有的话
		 */
		String getFactoryMethodName();
	
		/**
		 * 指定工厂方法（如果有）。 该方法将使用构造函数参数进行调用，如果没有指定，则不使用参数。 
		 * 该方法将在指定的工厂bean（如果有）或其他方式在本地bean类上作为静态方法调用。
		* @param factoryMethodName 静态工厂方法名称，如果正常的构造函数创建应该使用null
		 */
		void setFactoryMethodName(String factoryMethodName);
	
		/**
		 * 返回此bean的当前目标作用域的名称，如果不知道返回null
		 */
		String getScope();
	
		/**
		 * 覆盖当前的目标作用域
		 * @see #SCOPE_SINGLETON
		 * @see #SCOPE_PROTOTYPE
		 */
		void setScope(String scope);
	
		/**
		 * 返回这个bean是否应该被懒加载，即启动时不需要立即实例化。
		 * 仅适用于单例
		 */
		boolean isLazyInit();
	
		/**
		 * 设置这个bean是否需要懒加载
		 * 如果是false，那么bean将在启动时被bean factory实例化，bean factory会对单例立即初始化
		 */
		void setLazyInit(boolean lazyInit);
	
		/**
		 * 获取依赖的beanName
		 */
		String[] getDependsOn();
	
		/**
		 * 设置依赖的beanName
		 * bean factory将保证这些bean首先被初始化.
		 */
		void setDependsOn(String[] dependsOn);
	
		/**
		 * 返回这个bean是否是自动关联到其他bean。
		 */
		boolean isAutowireCandidate();
	
		/**
		 * 设置这个bean是否自动关联到其他的bean.
		 */
		void setAutowireCandidate(boolean autowireCandidate);
	
		/**
		 * 返回该bean是否是主要的关联着。 如果这个值对于多个匹配bean中的一个bean是真实的，则它将作为一个主要关联者。
		 */
		boolean isPrimary();
	
		/**
		 * 设置这个bean是否是为主要的关联者。
		 * 如果这个值对于多个匹配bean中的一个bean是真实的，则它将用作主要关联者
		 */
		void setPrimary(boolean primary);
	
	
		/**
		 * 返回此bean的构造函数的参数值。
		 * 可以在bean factory 处理期间修改返回的实例。
		 * @返回ConstructorArgumentValues对象（never  null）
		 */
		ConstructorArgumentValues getConstructorArgumentValues();
	
		/**
		 * 返回要应用于bean的新实例的属性值.
		 * 可以在bean factory 处理期间修改返回的实例。
		 * @return the MutablePropertyValues object (never <code>null</code>)
		 */
		MutablePropertyValues getPropertyValues();
	
	
		/**
		 * 返回是否是一个单例，如果是一个单例，在所有调用上返回同一个实例。
		 */
		boolean isSingleton();
	
		/**
		 * 返回这是否为原型，如果是每个调用返回一个独立的实例。
		 * @see #SCOPE_PROTOTYPE
		 */
		boolean isPrototype();
	
		/**
		 * 返回这个bean是否是“抽象的”，也就是，不要进行实例化。
		 */
		boolean isAbstract();
	
		/**
		 * 获取此BeanDefinition的角色提示。 角色提示为工具提供了特定BeanDefinition的重要性
		 * @see #ROLE_APPLICATION
		 * @see #ROLE_INFRASTRUCTURE
		 * @see #ROLE_SUPPORT
		 */
		int getRole();
	
		/**
		 * 返回这个bean定义的可读的描述。
		 */
		String getDescription();
	
		/**
		 *返回此bean定义来源的资源的描述（用于出现错误情况下显示上下文）。
		 */
		String getResourceDescription();
	
		/**
		 * 返回原始BeanDefinition，不存在返回null。
		 * 允许检索装饰的bean定义（如果有）。
		 * 请注意，此方法返回当前发起者。通过发起者链迭代，以查找用户定义的原始BeanDefinition。
		 */
		BeanDefinition getOriginatingBeanDefinition();
	
	}

从源码来看这里和依赖关系相关的方法主要是 **getDependsOn()**和**setDependsOn()**，也是在初始化过程中把读取到的依赖关系通过set方法设置到BeanDefinition中保存的。


# spring IOC 流程 #
**流程概述**

在使用spring IOC的情况下，需要什么信息才能创建bean呢？

1. 需要持有各种bean的定义
2. 需要持有bean之间的依赖关系
3. 需要一个工具完成上述需求的读取

前面介绍BeanFactory的时候我们说到，BeanFactory的默认实现是DefaultListableBeanFactory，我们参看源码可以看到在DefaultListableBeanFactory中持有一个BeanDefinition的集合，该集合记录了从Resource定位、载入和注册的bean的定义和依赖关系。

	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();

> 总结的来说：
> 
> - 需要一个file指向我们的xml：资源定位，指到XML的位置
> - 需要一个Reader读取XML： DOM解析
> - 需要将解析出来的数据设置到beanDefinitionMap中。


**ClasspathXmlApplicationContext解析**

## 整体看继承实现关系 ##
ClasspathXmlApplicationContext是ApplicationContext容器继承体系中的一个具体实现，该体系中还包含XmlWebApplicationContext和FileSystemXmlApplicationContext等，其继承体系如下图所示：
![XmlWebApplicationContext设计架构图](https://github.com/gamesdoa/img0/raw/master/spring/ApplicationContext.jpg)

> - 图中关注标红的对象ApplicationContext基本接口；
> - ClasspathXmlApplicationContext这个类是从classpath下加载配置文件(适合于相对路径方式加载)；
> - XmlWebApplicationContext专为web工程定制的方法，推荐Web项目中使用；
> - FileSystemXmlApplicationContext这个类是从文件绝对路径加载配置文件。

> 注：ApplicationContext允许上下文嵌套，通过保持父上下文可以维持一个上下文体系。对于bean的查找可以在这个上下文体系中发生，首先检查当前上下文，其次是父上下文，逐级向上，这样为不同的Spring应用提供了一个共享的bean定义环境。(在BeanFactory源码中获取bean的方法注释中有提到过)


## ClasspathXmlApplicationContext的IOC容器流程 ##

### 初始化 ###

	//该方法参数中classpath: 前缀是不需要的，默认就是指项目的classpath路径下面；
	//这也就是说用ClassPathXmlApplicationContext时默认的根目录是在WEB-INF/classes下面，而不是项目根目录。
	ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationcontext.xml");

### ClassPathXmlApplicationContext完整源码 ###

下面我们来看一下ClassPathXmlApplicationContext源码中都干了些什么事情？

	package org.springframework.context.support;
	
	import org.springframework.beans.BeansException;
	import org.springframework.context.ApplicationContext;
	import org.springframework.core.io.ClassPathResource;
	import org.springframework.core.io.Resource;
	import org.springframework.util.Assert;

	public class ClassPathXmlApplicationContext extends AbstractXmlApplicationContext {
	
		private Resource[] configResources;
	
		/**
		 * 用bean-style的配置创建一个新的ClassPathXmlApplicationContext.
		 */
		public ClassPathXmlApplicationContext() {
		}
	
		/**
		 * 用bean-style的配置创建一个新的ClassPathXmlApplicationContext.
		 * @param parent 父上下文
		 */
		public ClassPathXmlApplicationContext(ApplicationContext parent) {
			super(parent);
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件加载definitions并自动刷新上下文。
		 * @param configLocation 资源位置
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
			this(new String[] {configLocation}, true, null);
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件加载definitions并自动刷新上下文。
		 * @param configLocations 资源位置数组
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
			this(configLocations, true, null);
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件加载definitions并自动刷新上下文。
		 * @param configLocations 资源位置数组
		 * @param parent 父上下文
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
			this(configLocations, true, parent);
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件中加载定义。
		 * @param configLocations 资源位置数组
		 * @param refresh 是否自动刷新上下文，加载所有bean定义并创建所有单例。或者，在进一步配置上下文之后手动调用刷新。
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
			this(configLocations, refresh, null);
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件中加载定义。
		 * @param configLocations 资源位置数组
		 * @param refresh 是否自动刷新上下文，加载所有bean定义并创建所有单例。或者，在进一步配置上下文之后手动调用刷新。
		 * @param parent 父上下文
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
			super(parent);
			setConfigLocations(configLocations);
			if (refresh) {
				refresh();
			}
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件加载definitions并自动刷新上下文。
		 * 这是一个加载资源路径相对于给定类的方法。
		 * 为了充分的灵活性，请考虑使用GenericApplicationContext与XmlBeanDefinitionReader和ClassPathResource参数。
		 * @param path 类路径内的相对（或绝对）路径
		 * @param clazz 加载资源的类（给定基础的路径）
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String path, Class clazz) throws BeansException {
			this(new String[] {path}, clazz);
		}
	
		/**
		 * 创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件加载definitions并自动刷新上下文。
		 * @param paths 类路径内的相对（或绝对）路径数组
		 * @param clazz 加载资源的类（给定基础的路径）
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String[] paths, Class clazz) throws BeansException {
			this(paths, clazz, null);
		}
	
		/**
		 * 根据给定的路径创建一个新的ClassPathXmlApplicationContext，并从给定的XML文件加载definitions并自动刷新上下文。
		 * @param paths 类路径内的相对（或绝对）路径数组
		 * @param clazz 加载资源的类（给定基础的路径）
		 * @param parent 父上下文
		 * @throws BeansException 如果创建失败抛出异常
		 */
		public ClassPathXmlApplicationContext(String[] paths, Class clazz, ApplicationContext parent)
				throws BeansException {
			super(parent);
			Assert.notNull(paths, "Path array must not be null");
			Assert.notNull(clazz, "Class argument must not be null");
			this.configResources = new Resource[paths.length];
			for (int i = 0; i < paths.length; i++) {
				this.configResources[i] = new ClassPathResource(paths[i], clazz);
			}
			refresh();
		}
	
		@Override
		protected Resource[] getConfigResources() {
			return this.configResources;
		}
	}

### 解析初始化 ###

回到我们刚刚的初始化部分，将会调用构造函数：

	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}

该构造函数实际上调用

	public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

> 通过分析ClassPathXmlApplicationContext的源代码和我们单独提出来的初始化方法可以知道，在创建ClassPathXmlApplicationContext容器时，构造方法做以下三项工作：
> 
> - 用父类容器的构造方法(super(parent)方法)为容器设置好Bean资源加载器。主要是设置了下面这三个属性
> > this.parent = parent;//父上下文
> 
> > this.resourcePatternResolver = getResourcePatternResolver();//资源解析器
> 
> > this.environment = this.createEnvironment();//环境
> 
> - 调用父类AbstractRefreshableConfigApplicationContext的setConfigLocations(configLocations)方法设置Bean定义的资源文件的路径。
> - 是否立刻刷新上下文，如果不立刻刷新的话可以在进一步配置上下文之后手动调用刷新。

#### 设置Bean资源加载器 ####
接下来看一下ClassPathXmlApplicationContext的父类AbstractApplicationContext在初始化IoC容器中会做哪些事情，也就是设置Bean资源加载器这里，我们可以从架构图上看到AbstractApplicationContext并不是ClassPathXmlApplicationContext的直接父类，而是四层父类，中间的父类在初始化的时候只是调用了super(parent)方法，具体的处理还是在AbstractApplicationContext中，这部分的主要源码如下：
	
	public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext, DisposableBean {  
	    //静态初始化块，在整个容器创建过程中只执行一次  
	    static {  
	        //为了避免应用程序在Weblogic8.1关闭时出现类加载异常加载问题，加载IoC容器关闭事件(ContextClosedEvent)类  
	        ContextClosedEvent.class.getName();  
	    }
		……
	    //ClassPathXmlApplicationContextt调用父类构造方法调用的就是该方法  
	    public AbstractApplicationContext(ApplicationContext parent) {  
	        this.parent = parent;
			this.resourcePatternResolver = getResourcePatternResolver();
			this.environment = this.createEnvironment();
	    }  
	    //获取一个Spring Source的加载器用于读入Spring Bean定义资源文件  
		protected ResourcePatternResolver getResourcePatternResolver() {
			//AbstractApplicationContext继承DefaultResourceLoader，也是一个Spring资源加载器，其getResource(String location)方法用于载入资源 
			return new PathMatchingResourcePatternResolver(this);
		} 
		……  
	}


AbstractApplicationContext构造方法中调用的getResourcePatternResolver()方法中创建了一个新的PathMatchingResourcePatternResolver创建Spring资源加载器：

	public class PathMatchingResourcePatternResolver implements ResourcePatternResolver {
		……
		public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {
			Assert.notNull(resourceLoader, "ResourceLoader must not be null");
			this.resourceLoader = resourceLoader;
		}
		……
	}

#### 定位bean定义的资源文件 ####
在设置完容器的资源加载器之后，接下来ClassPathXmlApplicationContext执行setConfigLocations方法通过调用其父类AbstractRefreshableConfigApplicationContext的方法进行对指定Bean定义的资源文件的定位，该方法的源码如下：

	public abstract class AbstractRefreshableConfigApplicationContext extends AbstractRefreshableApplicationContext 
		implements BeanNameAware, InitializingBean {
		……
		
		/**
		 * 在init-param style中设置此应用程序上下文的配置位置，即用逗号，分号或空格分隔的不同位置。
		 * 如果未设置，则实施可能会使用默认值。
		 */
		public void setConfigLocation(String location) {
			setConfigLocations(StringUtils.tokenizeToStringArray(location, CONFIG_LOCATION_DELIMITERS));
		}
		
		/**
		 * 设置此应用程序上下文的配置位置.
		 * 如果未设置，则使用时可能会使用默认值。
		 */
		public void setConfigLocations(String[] locations) {
			if (locations != null) {
				Assert.noNullElements(locations, "Config locations must not be null");
				this.configLocations = new String[locations.length];
				for (int i = 0; i < locations.length; i++) {
					//resolvePath为同一个类中将字符串解析为路径的方法
					this.configLocations[i] = resolvePath(locations[i]).trim();
				}
			}
			else {
				this.configLocations = null;
			}
		}
		……
	}

> 通过这两个方法的源码我们可以看出，我们既可以使用一个字符串来配置多个Spring Bean定义文件，也可以使用字符串数组，即下面两种方式都是可以的：
> 
> - ClasspathResource res = new ClasspathResource("a.xml,b.xml,……");  多个资源文件路径之间可以是用",;/t/n"等分隔。
> - ClasspathResource res = new ClasspathResource(newString[]{"a.xml","b.xml",……});

到这里，Spring IoC容器在初始化时将配置的Bean定义文件定位为Spring封装的Resource的工作就完成了。

#### refresh方法 ####
我们说过初始化过程包含Resource定位、载入和Bean注册三个步骤，各个我们学习了Resource定位问题，下面我们来看一下载入这一步。
我们看到在初始化的第三步判断是否立刻刷新上下文，这里就是载入的入口部分了，也就是refresh()方法，refresh()是一个模板方法。
##### refresh源码 #####
ClassPathXmlApplicationContext通过调用其父类AbstractApplicationContext的refresh()函数启动整个IoC容器对Bean定义的载入过程：

	public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext, DisposableBean {
		……
		public void refresh() throws BeansException, IllegalStateException {
			synchronized (this.startupShutdownMonitor) {
				// 调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识  
				prepareRefresh();

				// 告诉子类启动refreshBeanFactory()方法，Bean定义资源文件的载入从子类的refreshBeanFactory()方法启动  
				ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
	
				// 为BeanFactory配置容器特性，例如类加载器、事件处理器等 
				prepareBeanFactory(beanFactory);
	
				try {
					// 为容器的某些子类指定特殊的BeanPost事件处理器  
					postProcessBeanFactory(beanFactory);
	
					//调用所有该上下文中注册的BeanFactoryPostProcessor的Bean 
					invokeBeanFactoryPostProcessors(beanFactory);
	
					// 为BeanFactory注册BeanPost事件处理器. 
					// BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件  
					registerBeanPostProcessors(beanFactory);
	
					// 初始化该上下文的信息源，和国际化相关.
					initMessageSource();
	
					// 初始化该上下文容器事件传播器.
					initApplicationEventMulticaster();
	
					// 在特定的上下文子类中初始化其他特殊的bean.
					onRefresh();
	
					// 检查监听器bean并注册它们.
					registerListeners();
	
					// 实例化所有剩下的（非lazy-init）单例.
					finishBeanFactoryInitialization(beanFactory);
	
					// 最后一步：发布相应的事件.
					finishRefresh();
				} catch (BeansException ex) {
					//销毁已经创建的单例，以避免资源悬空.
					destroyBeans();
					// 重置 'active' 标识.
					cancelRefresh(ex);
					// 抛出异常给调用者.
					throw ex;
				}
			}
		}
		……
		/**
		 * 告诉子类启动refreshBeanFactory()方法，
		 * @return 刷新的BeanFactory实例
		 */
		protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
			//这里使用了委派设计模式，父类定义了抽象的refreshBeanFactory()方法，具体实现调用子类容器的refreshBeanFactory()方法
			refreshBeanFactory();
			ConfigurableListableBeanFactory beanFactory = getBeanFactory();
			if (logger.isDebugEnabled()) {
				logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
			}
			return beanFactory;
		}
		……
		protected abstract void refreshBeanFactory() throws BeansException, IllegalStateException;
		……
	}

Spring IoC容器载入Bean定义的资源文件从其子类容器的refreshBeanFactory()方法启动，所以整个refresh()中“ConfigurableListableBeanFactory beanFactory =obtainFreshBeanFactory();”这句以后代码的都是注册容器的信息源和生命周期事件，载入过程就是从这句代码启动。

refresh()方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。refresh的作用类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入。
##### refreshBeanFactory源码 #####
- AbstractApplicationContext类中只抽象定义了refreshBeanFactory()方法，容器真正调用的是其子类AbstractRefreshableApplicationContext实现的refreshBeanFactory()方法，

方法的源码如下：
	
	public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {
		……
		/**
		 * 此实现将对该上下文的底层BeanFactory进行实际刷新，关闭以前的BeanFactory（如果有），
		 * 并为上下文生命周期的下一阶段初始化一个新的BeanFactory。
		 */
		@Override
		protected final void refreshBeanFactory() throws BeansException {
			if (hasBeanFactory()) {
				//如果已经有容器，销毁容器中的bean，关闭容器 
				destroyBeans();
				closeBeanFactory();
			}
			try {
				//创建新的IoC容器也就是BeanFactory
				DefaultListableBeanFactory beanFactory = createBeanFactory();
				beanFactory.setSerializationId(getId());
				//对IoC容器进行定制化，如设置启动参数，开启注解的自动装配等 
				customizeBeanFactory(beanFactory);
				//调用载入Bean定义的方法，这里又使用了委派模式，在当前类中只定义了抽象的loadBeanDefinitions方法，具体的实现调用子类容器
				loadBeanDefinitions(beanFactory);
				synchronized (this.beanFactoryMonitor) {
					this.beanFactory = beanFactory;
				}
			}
			catch (IOException ex) {
				throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
			}
		}
		……
		/**
		 * 通常通过委托一个或多个bean definition readers将beande finition加载到给定的bean工厂中。
		 * @param beanFactory 用于加载bean definitions的beanFactory
		 * @throws BeansException 如果解析bean definitions失败
		 * @throws IOException if 如果加载bean definitions文件失败
		 */
		protected abstract void loadBeanDefinitions(DefaultListableBeanFactory beanFactory)
				throws BeansException, IOException;
		……
	}

> 在这个方法中，先判断BeanFactory是否存在，如果存在则先销毁beans并关闭beanFactory，接着创建DefaultListableBeanFactory，并调用loadBeanDefinitions(beanFactory)装载bean定义。
##### loadBeanDefinitions源码 #####
- AbstractRefreshableApplicationContext中只定义了抽象的loadBeanDefinitions()方法，容器真正调用的是其子类AbstractXmlApplicationContext对该方法的实现.

主要源码如下：
	
	public abstract class AbstractXmlApplicationContext extends AbstractRefreshableConfigApplicationContext {
		……
		/**
		 * 通过 XmlBeanDefinitionReader 加载 bean definitions.
		 */
		@Override
		protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
			// 根据给定的BeanFactory创建一个新的XmlBeanDefinitionReader.
			XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
	
			// 根据上下文资源环境配置bean definition reader的环境.
			beanDefinitionReader.setEnvironment(this.getEnvironment());

			//为Bean读取器设置Spring资源加载器，AbstractXmlApplicationContext的  
			//祖先父类AbstractApplicationContext继承DefaultResourceLoader，因此，容器本身也是一个资源加载器 
			beanDefinitionReader.setResourceLoader(this);

			//为Bean读取器设置SAX xml解析器 
			beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
	
			// 允许子类提供reader的自定义初始化，然后继续实现加载bean definitions。
			initBeanDefinitionReader(beanDefinitionReader);

			//Bean读取器真正实现加载的方法
			loadBeanDefinitions(beanDefinitionReader);
		}
		……
		/**
		 * 使用给定的XmlBeanDefinitionReader加载bean definitions。
		 * bean工厂的生命周期由#refreshBeanFactory方法处理; 因此这个方法只是加载和/或注册bean定义。
		 * @param reader the XmlBeanDefinitionReader to use
		 * @throws BeansException in case of bean registration errors
		 * @throws IOException if the required XML document isn't found
		 */
		protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
			//获取Bean定义资源的定位
			Resource[] configResources = getConfigResources();
			if (configResources != null) {
				//Xml Bean读取器调用其父类AbstractBeanDefinitionReader读取定位的Bean定义资源  
				reader.loadBeanDefinitions(configResources);
			}

			//如果子类中获取的Bean定义资源定位为空，则获取FileSystemXmlApplicationContext构造方法中setConfigLocations方法设置的资源
			String[] configLocations = getConfigLocations();
			if (configLocations != null) {
				//Xml Bean读取器调用其父类AbstractBeanDefinitionReader读取定位的Bean定义资源 
				reader.loadBeanDefinitions(configLocations);
			}
		}
		……
		/**
		 * 返回一个资源对象数组，引用这个上下文应该构建的XML bean定义文件。
		 * 默认实现返回null。 子类可以覆盖此提供预先构建的资源对象而不是位置字符串。
		 * 这里使用了委托模式，调用子类的获取Bean定义资源定位的方法，该方法在ClassPathXmlApplicationContext中进行实现，
		 * FileSystemXmlApplicationContext中并没有实现
		 * @return an array of Resource objects, or <code>null</code> if none
		 * @see #getConfigLocations()
		 */
		protected Resource[] getConfigResources() {
			return null;
		}
		……
	}

> Xml Bean读取器(XmlBeanDefinitionReader)调用其父类AbstractBeanDefinitionReader的reader.loadBeanDefinitions方法读取Bean定义资源。

ClassPathXmlApplicationContext中重写了getConfigResources()方法，返回configResources的属性的值。

	public class ClassPathXmlApplicationContext extends AbstractXmlApplicationContext {
		……
		@Override
		protected Resource[] getConfigResources() {
			return this.configResources;
		}
		……
	}

然后根据情况判断是否执行reader.loadBeanDefinitions(configResources)分支，之后再获取getConfigLocations(),判断是否执行reader.loadBeanDefinitions(configLocations)分支。

##### 读取Bean定义资源 #####

读取Bean定义资源的实际工作是在其抽象父类AbstractBeanDefinitionReader中定义的，主要源码：

	public abstract class AbstractBeanDefinitionReader implements EnvironmentCapable, BeanDefinitionReader {
		……

		//重载方法，调用下面的loadBeanDefinitions(String, Set<Resource>);方法 
		public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
			return loadBeanDefinitions(location, null);
		}
	
		/**
		 * 从指定的资源位置加载bean定义。
		 * 该位置也可以是一个location pattern，前提是该bean定义阅读器的ResourceLoader是ResourcePatternResolver。
		 * @param location 要加载这个bean定义阅读器的ResourceLoader（或ResourcePatternResolver）
		 * @param actualResources 要在加载过程中解析的实际资源对象填充的集合。可能是null来表示调用者对这些资源对象不感兴趣。
		 * @return the number of bean definitions found
		 * @throws BeanDefinitionStoreException in case of loading or parsing errors
		 */
		public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
			//获取在IoC容器初始化过程中设置的资源加载器  
			ResourceLoader resourceLoader = getResourceLoader();
			if (resourceLoader == null) {
				throw new BeanDefinitionStoreException(
						"Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
			}
	
			if (resourceLoader instanceof ResourcePatternResolver) {
				// 可以用资源模式匹配.
				try {
					//将指定位置的Bean定义资源文件解析为Spring IoC容器封装的资源加载多个指定位置的Bean定义资源文件  
					Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
					//委派调用其子类XmlBeanDefinitionReader的方法，实现加载功能 
					int loadCount = loadBeanDefinitions(resources);
					if (actualResources != null) {
						for (Resource resource : resources) {
							actualResources.add(resource);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
					}
					return loadCount;
				}
				catch (IOException ex) {
					throw new BeanDefinitionStoreException(
							"Could not resolve bean definition resource pattern [" + location + "]", ex);
				}
			}
			else {
				//只能通过绝对URL加载单个资源。
				Resource resource = resourceLoader.getResource(location);
				//委派调用其子类XmlBeanDefinitionReader的方法，实现加载功能
				int loadCount = loadBeanDefinitions(resource);
				if (actualResources != null) {
					actualResources.add(resource);
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
				}
				return loadCount;
			}
		}

		//重载方法，调用loadBeanDefinitions(String); 
		public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
			Assert.notNull(locations, "Location array must not be null");
			int counter = 0;
			for (String location : locations) {
				counter += loadBeanDefinitions(location);
			}
			return counter;
		}
		……
	}

> loadBeanDefinitions(Resource...resources)方法和上面分析的3个方法类似，同样也是调用XmlBeanDefinitionReader的loadBeanDefinitions方法。从对AbstractBeanDefinitionReader的loadBeanDefinitions方法源码分析可以看出该方法做了以下两件事：
> 
> - 调用资源加载器的获取资源方法resourceLoader.getResource(location)，获取到要加载的资源。
> - 真正执行加载功能是其子类XmlBeanDefinitionReader的loadBeanDefinitions方法。

> 通过ResourceLoader resourceLoader = getResourceLoader(); 和
> 
> Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
> 
> 可以知道此时调用的是DefaultResourceLoader中的getSource()方法定位Resource

##### 获取要读入的资源 #####
XmlBeanDefinitionReader通过调用其父类DefaultResourceLoader的getResource方法获取要加载的资源，其源码如下

	public class DefaultResourceLoader implements ResourceLoader {
		……
		public Resource getResource(String location) {
			Assert.notNull(location, "Location must not be null");
			//是不是classpath:，也就是相对路径
			if (location.startsWith(CLASSPATH_URL_PREFIX)) {
				return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
			}
			else {
				try {
					// 尝试将该位置解析为URL...
					URL url = new URL(location);
					return new UrlResource(url);
				}
				catch (MalformedURLException ex) {
					// 没有URL - >解析为资源路径。
					return getResourceByPath(location);
				}
			}
		}
		……
	}

我们解析的 ClassPathXmlApplicationContext 符合location.startsWith(CLASSPATH_URL_PREFIX) 所以新建一个ClassPathResource返回，如果是FileSystemXmlApplicationContext的话，因为是绝对路径是应该是调用FileSystemXmlApplicationContext的getResourceByPath()方法返回Resource。

##### 加载Bean定义资源 #####

Bean定义的Resource得到了，继续回到XmlBeanDefinitionReader的loadBeanDefinitions(Resource …)方法看到代表bean文件的资源定义以后的载入过程。

	public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {
		/**
		 * 从指定的XML文件加载bean定义。
		 * @param resource the resource descriptor for the XML file
		 * @return the number of bean definitions found
		 * @throws BeanDefinitionStoreException in case of loading or parsing errors
		 */
		public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
			//将读入的XML资源进行特殊编码处理 
			return loadBeanDefinitions(new EncodedResource(resource));
		}
	
		/**
		 * 从指定的XML文件加载bean定义。
		 * @param encodedResource  XML文件的资源描述符，允许指定用于解析文件的编码
		 * @return the number of bean definitions found
		 * @throws BeanDefinitionStoreException in case of loading or parsing errors
		 */
		public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
			Assert.notNull(encodedResource, "EncodedResource must not be null");
			if (logger.isInfoEnabled()) {
				logger.info("Loading XML bean definitions from " + encodedResource.getResource());
			}
	
			Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
			if (currentResources == null) {
				currentResources = new HashSet<EncodedResource>(4);
				this.resourcesCurrentlyBeingLoaded.set(currentResources);
			}
			if (!currentResources.add(encodedResource)) {
				throw new BeanDefinitionStoreException(
						"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
			}
			try {
				//将资源文件转为InputStream的IO流 
				InputStream inputStream = encodedResource.getResource().getInputStream();
				try {
					//从InputStream中得到XML的解析源 
					InputSource inputSource = new InputSource(inputStream);
					if (encodedResource.getEncoding() != null) {
						inputSource.setEncoding(encodedResource.getEncoding());
					}
					//这里是具体的读取过程  
					return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
				}
				finally {
					//关闭从Resource中得到的IO流   
					inputStream.close();
				}
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"IOException parsing XML document from " + encodedResource.getResource(), ex);
			}
			finally {
				currentResources.remove(encodedResource);
				if (currentResources.isEmpty()) {
					this.resourcesCurrentlyBeingLoaded.remove();
				}
			}
		}
		/**
		 * //从特定XML文件中实际载入Bean定义资源的方法 
		 * @param inputSource the SAX InputSource to read from
		 * @param resource the resource descriptor for the XML file
		 * @return the number of bean definitions found
		 * @throws BeanDefinitionStoreException in case of loading or parsing errors
		 */
		protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
				throws BeanDefinitionStoreException {
			try {
				int validationMode = getValidationModeForResource(resource);
				//将XML文件转换为DOM对象，解析过程由documentLoader实现  
				Document doc = this.documentLoader.loadDocument(
						inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
				//这里是启动对Bean定义解析的详细过程，该解析过程会用到Spring的Bean配置规则
				return registerBeanDefinitions(doc, resource);
			}
			……
		}
	}

> 通过源码分析，载入Bean定义资源文件的最后一步是将Bean定义资源转换为Document对象，该过程由documentLoader实现

##### 换为Document对象 #####

	public class DefaultDocumentLoader implements DocumentLoader {
		//使用标准的JAXP将载入的Bean定义资源转换成document对象
		public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
				ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {
			//创建文件解析器工厂 
			DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
			if (logger.isDebugEnabled()) {
				logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
			}
			//创建文档解析器
			DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
			//解析Spring的Bean定义资源
			return builder.parse(inputSource);
		}
	
		protected DocumentBuilderFactory createDocumentBuilderFactory(int validationMode, boolean namespaceAware)
				throws ParserConfigurationException {
			//创建文档解析工厂 
			DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
			factory.setNamespaceAware(namespaceAware);
			//设置解析XML的校验
			if (validationMode != XmlValidationModeDetector.VALIDATION_NONE) {
				factory.setValidating(true);
				if (validationMode == XmlValidationModeDetector.VALIDATION_XSD) {
					factory.setNamespaceAware(true);
					try {
						factory.setAttribute(SCHEMA_LANGUAGE_ATTRIBUTE, XSD_SCHEMA_LANGUAGE);
					}
					catch (IllegalArgumentException ex) {
						ParserConfigurationException pcex = new ParserConfigurationException(
								"Unable to validate using XSD: Your JAXP provider [" + factory +
								"] does not support XML Schema. Are you running on Java 1.4 with Apache Crimson? " +
								"Upgrade to Apache Xerces (or Java 1.5) for full XSD support.");
						pcex.initCause(ex);
						throw pcex;
					}
				}
			}
			return factory;
		}
	}

> 该解析过程调用JavaEE标准的JAXP标准进行处理。

至此Spring IoC容器根据定位的Bean定义资源文件，将其加载读入并转换成为Document对象过程完成。

 
##### 解析Bean定义资源 #####

XmlBeanDefinitionReader类中的doLoadBeanDefinitions方法是从特定XML文件中实际载入Bean定义资源的方法，该方法在载入Bean定义资源之后将其转换为Document对象，接下来调用registerBeanDefinitions启动Spring IoC容器对Bean定义的解析过程，registerBeanDefinitions方法源码如下：

	public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {
		//按照Spring的Bean语义要求将Bean定义资源解析并转换为容器内部数据结构
		public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
			//得到BeanDefinitionDocumentReader来对xml格式的BeanDefinition解析 
			BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
			documentReader.setEnvironment(this.getEnvironment());
			//获得容器中注册的Bean数量
			int countBefore = getRegistry().getBeanDefinitionCount();
			
			//解析过程入口，这里使用了委派模式，BeanDefinitionDocumentReader只是个接口，
			//具体的解析实现过程有实现类DefaultBeanDefinitionDocumentReader完成  
			documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
			//统计解析的Bean数量
			return getRegistry().getBeanDefinitionCount() - countBefore;
		}

		//创建BeanDefinitionDocumentReader对象，解析Document对象
		protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {
			return BeanDefinitionDocumentReader.class.cast(BeanUtils.instantiateClass(this.documentReaderClass));
		}
	}

> Bean定义资源的载入解析分为以下两个过程：
>
> - 通过调用XML解析器将Bean定义资源文件转换得到Document对象，但是这些Document对象并没有按照Spring的Bean规则进行解析。这一步是载入的过程
> - 在完成通用的XML解析之后，按照Spring的Bean规则对Document对象进行解析。


##### 对Document解析 #####

BeanDefinitionDocumentReader接口通过registerBeanDefinitions方法调用其实现类DefaultBeanDefinitionDocumentReader对Document对象进行解析，解析的代码如下：

	public class DefaultBeanDefinitionDocumentReader implements BeanDefinitionDocumentReader {
		public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
			this.readerContext = readerContext;
	
			logger.debug("Loading bean definitions");
			Element root = doc.getDocumentElement();
	
			doRegisterBeanDefinitions(root);
		}
	
		/**
		 * 在给定的根{<beans />元素中注册每个bean定义。
		 */
		protected void doRegisterBeanDefinitions(Element root) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				Assert.state(this.environment != null, "environment property must not be null");
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!this.environment.acceptsProfiles(specifiedProfiles)) {
					return;
				}
			}
	
			// 任何嵌套的<beans>元素将导致此方法中的递归。 
			// 为了正确地传播和保存<beans> default- *属性，请跟踪当前（父）委托，它可能为null。
			// 创建一个引用父（parent）的新的（子）代理，以便进行回退，然后最终将this.delegate重置为原始（父）引用。 
			// 这种行为模拟一堆代理，而不需要一个代理。
			BeanDefinitionParserDelegate parent = this.delegate;
			this.delegate = createHelper(readerContext, root, parent);
	
			preProcessXml(root);
			parseBeanDefinitions(root, this.delegate);
			postProcessXml(root);
	
			this.delegate = parent;
		}
		protected BeanDefinitionParserDelegate createHelper(XmlReaderContext readerContext, Element root, BeanDefinitionParserDelegate parentDelegate) {
			BeanDefinitionParserDelegate delegate = new BeanDefinitionParserDelegate(readerContext, environment);
			delegate.initDefaults(root, parentDelegate);
			return delegate;
		}
	
		protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
			if (delegate.isDefaultNamespace(root)) {
				NodeList nl = root.getChildNodes();
				for (int i = 0; i < nl.getLength(); i++) {
					Node node = nl.item(i);
					if (node instanceof Element) {
						Element ele = (Element) node;
						if (delegate.isDefaultNamespace(ele)) {
							parseDefaultElement(ele, delegate);
						}
						else {
							delegate.parseCustomElement(ele);
						}
					}
				}
			}
			else {
				delegate.parseCustomElement(root);
			}
		}
	
		private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
			if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
				importBeanDefinitionResource(ele);
			} else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
				processAliasRegistration(ele);
			} else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
				processBeanDefinition(ele, delegate);
			} else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
				// 递归
				doRegisterBeanDefinitions(ele);
			}
		}
		
		protected void importBeanDefinitionResource(Element ele) {
			String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
			if (!StringUtils.hasText(location)) {
				getReaderContext().error("Resource location must not be empty", ele);
				return;
			}
	
			// 解决系统属性：例如“${user.dir}”
			location = environment.resolveRequiredPlaceholders(location);
	
			Set<Resource> actualResources = new LinkedHashSet<Resource>(4);
	
			// 发现位置是绝对的还是相对的URI
			boolean absoluteLocation = false;
			try {
				absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
			} catch (URISyntaxException ex) {
				// 不能转换为URI，考虑到位置相对，除非是众所周知的Spring前缀“classpath *：”
			}
	
			// 绝对还是相对？
			if (absoluteLocation) {
				try {
					int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
					if (logger.isDebugEnabled()) {
						logger.debug("Imported " + importCount + " bean definitions from URL location [" + location + "]");
					}
				} catch (BeanDefinitionStoreException ex) {
					getReaderContext().error(
							"Failed to import bean definitions from URL location [" + location + "]", ele, ex);
				}
			}
			else {
				// 没有URL - >考虑相对于当前文件的资源位置。
				try {
					int importCount;
					Resource relativeResource = getReaderContext().getResource().createRelative(location);
					if (relativeResource.exists()) {
						importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
						actualResources.add(relativeResource);
					} else {
						String baseLocation = getReaderContext().getResource().getURL().toString();
						importCount = getReaderContext().getReader().loadBeanDefinitions(
								StringUtils.applyRelativePath(baseLocation, location), actualResources);
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Imported " + importCount + " bean definitions from relative location [" + location + "]");
					}
				} catch (IOException ex) {
					getReaderContext().error("Failed to resolve current resource location", ele, ex);
				} catch (BeanDefinitionStoreException ex) {
					getReaderContext().error("Failed to import bean definitions from relative location [" + location + "]", ele, ex);
				}
			}
			Resource[] actResArray = actualResources.toArray(new Resource[actualResources.size()]);
			getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
		}
	
		/**
		 * 处理给定的别名元素，使用注册表注册别名。
		 */
		protected void processAliasRegistration(Element ele) {
			String name = ele.getAttribute(NAME_ATTRIBUTE);
			String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
			boolean valid = true;
			if (!StringUtils.hasText(name)) {
				getReaderContext().error("Name must not be empty", ele);
				valid = false;
			}
			if (!StringUtils.hasText(alias)) {
				getReaderContext().error("Alias must not be empty", ele);
				valid = false;
			}
			if (valid) {
				try {
					getReaderContext().getRegistry().registerAlias(name, alias);
				}
				catch (Exception ex) {
					getReaderContext().error("Failed to register alias '" + alias +
							"' for bean with name '" + name + "'", ele, ex);
				}
				getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
			}
		}
	
		/**
		 * 处理给定的bean元素，解析bean定义并将其注册到注册表。
		 */
		protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
			BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
			if (bdHolder != null) {
				bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
				try {
					// 注册最终解析的实例.
					BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
				}
				catch (BeanDefinitionStoreException ex) {
					getReaderContext().error("Failed to register bean definition with name '" +
							bdHolder.getBeanName() + "'", ele, ex);
				}
				// 发送注册事件.
				getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
			}
		}
	
	}

> 通过上述Spring IoC容器对载入的Bean定义Document解析可以看出，我们使用Spring时，在Spring配置文件中可以使用<Import>元素来导入IoC容器所需要的其他资源，Spring IoC容器在解析时会首先将指定导入的资源加载进容器中。使用<Ailas>别名时，Spring IoC容器首先将别名元素所定义的别名注册到容器中。

> 对于既不是<Import>元素，又不是<Alias>元素的元素，即Spring配置文件中普通的<Bean>元素的解析由BeanDefinitionParserDelegate类的parseBeanDefinitionElement方法来实现。

##### BeanDefinitionParserDelegate #####
该类主要作用是用于解析XML定义的bean定义为BeanDefinition对象。

解析Bean定义资源文件中的<Bean>元素的方法：

	//解析<Bean>元素的入口  
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {}
	//解析Bean定义资源文件中的<Bean>元素，这个方法中主要处理<Bean>元素的id，name 和别名属性  
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) { }
	//详细对<Bean>元素中配置的Bean定义其他属性进行解析，由于上面的方法中已经对
	//Bean的id、name和别名等属性进行了处理，该方法中主要处理除这三个以外的其他属性数据  
	public AbstractBeanDefinition parseBeanDefinitionElement(Element ele, String beanName, BeanDefinition containingBean) { }

解析<property>元素的方法：

	//解析<Bean>元素中的<property>子元素  
	public void parsePropertyElements(Element beanEle, BeanDefinition bd) {}
	//解析<property>元素  
	public void parsePropertyElement(Element ele, BeanDefinition bd) {}
	//解析获取property值  
	public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) { } 

解析<property>元素的子元素的方法：
	
	//解析<property>元素中ref,value或者集合等子元素  
	public Object parsePropertySubElement(Element ele, BeanDefinition bd, String defaultValueType) {}

解析<list>子元素的方法：

	//解析<list>集合子元素  
	public List parseListElement(Element collectionEle, BeanDefinition bd) {}
	 //具体解析<list>集合元素，<array>、<list>和<set>都使用该方法解析  
	protected void parseCollectionElements(
			NodeList elementNodes, Collection<Object> target, BeanDefinition bd, String defaultElementType) {} 


经过对Spring Bean定义资源文件转换的Document对象中的元素解析，Spring IoC现在已经将XML形式定义的Bean定义资源文件转换为BeanDefinition，它是Bean定义资源文件中配置的POJO对象在Spring IoC容器中的映射，我们可以通过AbstractBeanDefinition为入口，对IoC容器进行索引、查询和操作。

通过Spring IoC容器对Bean定义资源的解析后，IoC容器大致完成了管理Bean对象的初始化过程，接下来需要向容器注册Bean定义信息才能全部完成IoC容器的初始化过程

##### IoC容器中的注册 #####
DefaultBeanDefinitionDocumentReader对Bean定义转换的Document对象解析的结果中最后调用processBeanDefinition()方法，其中就有调用BeanDefinitionReaderUtils的registerBeanDefinition方法向IoC容器注册解析的Bean，BeanDefinitionReaderUtils的注册的源码如下：

	//将解析的BeanDefinitionHold注册到容器中 
	public static void registerBeanDefinition(BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)  
    throws BeanDefinitionStoreException {  
        //获取解析的BeanDefinition的名称
         String beanName = definitionHolder.getBeanName();  
        //向IoC容器注册BeanDefinition 
        registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());  
        //如果解析的BeanDefinition有别名，向容器为其注册别名  
         String[] aliases = definitionHolder.getAliases();  
        if (aliases != null) {  
            for (String aliase : aliases) {  
                registry.registerAlias(beanName, aliase);  
            }  
        }  
	}



当调用BeanDefinitionReaderUtils向IoC容器注册解析的BeanDefinition时，真正完成注册功能的是DefaultListableBeanFactory。


##### 注册解析后的BeanDefinition #####

DefaultListableBeanFactory中使用一个HashMap的集合对象存放IoC容器中注册解析的BeanDefinition，向IoC容器注册的主要源码如下：


	//存储注册的俄BeanDefinition  
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();  
	//向IoC容器注册解析的BeanDefiniton  
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)  
           throws BeanDefinitionStoreException {  
       Assert.hasText(beanName, "Bean name must not be empty");  
       Assert.notNull(beanDefinition, "BeanDefinition must not be null");  
       //校验解析的BeanDefiniton  
       if (beanDefinition instanceof AbstractBeanDefinition) {  
           try {  
               ((AbstractBeanDefinition) beanDefinition).validate();  
           }  
           catch (BeanDefinitionValidationException ex) {  
               throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,  
                       "Validation of bean definition failed", ex);  
           }  
       }  
       //注册的过程中需要线程同步，以保证数据的一致性  
       synchronized (this.beanDefinitionMap) {  
           Object oldBeanDefinition = this.beanDefinitionMap.get(beanName);  
           //检查是否有同名的BeanDefinition已经在IoC容器中注册，如果已经注册，  
           //并且不允许覆盖已注册的Bean，则抛出注册失败异常  
           if (oldBeanDefinition != null) {  
               if (!this.allowBeanDefinitionOverriding) {  
                   throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,  
                           "Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +  
                           "': There is already [" + oldBeanDefinition + "] bound.");  
               }  
               else {//如果允许覆盖，则同名的Bean，后注册的覆盖先注册的  
                   if (this.logger.isInfoEnabled()) {  
                       this.logger.info("Overriding bean definition for bean '" + beanName +  
                               "': replacing [" + oldBeanDefinition + "] with [" + beanDefinition + "]");  
                   }  
               }  
           }  
           //IoC容器中没有已经注册同名的Bean，按正常注册流程注册  
           else {  
               this.beanDefinitionNames.add(beanName);  
               this.frozenBeanDefinitionNames = null;  
           }  
           this.beanDefinitionMap.put(beanName, beanDefinition);  
           //重置所有已经注册过的BeanDefinition的缓存  
           resetBeanDefinition(beanName);  
       }  
    }


至此，Bean定义资源文件中配置的Bean被解析并且注册到IoC容器中。现在IoC容器中已经存在了所有Bean的配置信息，这些BeanDefinition信息已经可以使用，并且可以被检索，IoC容器的作用就是对这些注册的Bean定义信息进行处理和维护。因为IOC容器持有这些BeanDefinition信息，所以可以根据描述进行bean的注入。


## 总结 ##

现在通过上面的代码，总结一下IOC容器初始化的基本步骤：

- 初始化的入口在容器实现中的 refresh()调用来完成

- 对 bean 定义载入 IOC 容器使用的方法是 loadBeanDefinition,
 > 其中的大致过程如下：
 > 
 > 通过 ResourceLoader 来完成资源文件位置的定位，DefaultResourceLoader 是默认的实现，同时上下文本身就给出了 ResourceLoader 的实现，可以从类路径，文件系统, URL 等方式来定为资源位置;
 > 
 > 如果是 XmlBeanFactory作为 IOC 容器，那么需要为它指定 bean 定义的资源，也就是说 bean 定义文件时通过抽象成 Resource 来被 IOC 容器处理的;
 > 
 > 容器通过 BeanDefinitionReader来完成定义信息的解析和 Bean 信息的注册,往往使用的是XmlBeanDefinitionReader 来解析 bean 的 xml 定义文件 (实际的处理过程是委托给 BeanDefinitionParserDelegate 来完成的)，从而得到 bean 的定义信息;
 > 
 > 这些信息在 Spring 中使用 BeanDefinition 对象来表示( 这个名字可以让我们想到loadBeanDefinition,RegisterBeanDefinition  这些相关的方法 ) 他们都是为处理 BeanDefinitin 服务的， 
 > 
 > 容器解析得到 BeanDefinitionIoC 以后，需要把它在 IOC 容器中注册，这由 IOC 实现 BeanDefinitionRegistry 接口来实现。
 > 
 > 注册过程就是在 IOC 容器内部维护的一个HashMap 来保存得到的 BeanDefinition 的过程。这个 HashMap 是 IoC 容器持有 bean 信息的场所，以后对 bean 的操作都是围绕这个HashMap 来实现的.

 

-----------------------
>在使用 Spring IOC 容器的时候我们还需要区别两个概念:Beanfactory 和 Factory bean，
>
> - BeanFactory 指的是 IOC 容器的编程抽象，比如 ApplicationContext， XmlBeanFactory 等，这些都是 IOC 容器的具体表现，需要使用什么样的容器由客户决定,但 Spring 为我们提供了丰富的选择。 
> - FactoryBean 只是一个可以在 IOC而容器中被管理的一个 bean,是对各种处理过程和资源使用的抽象,Factory bean 在需要时产生另一个对象，而不返回 FactoryBean本身,我们可以把它看成是一个抽象工厂，对它的调用返回的是工厂生产的产品。所有的 Factory bean 都实现特殊的org.springframework.beans.factory.FactoryBean 接口，当使用容器中 factory bean 的时候，返回其生成的对象。