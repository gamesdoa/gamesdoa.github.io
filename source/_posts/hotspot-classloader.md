---
title: hotspot类加载机制分析
date: 2017-06-10
tags: 
	- java
	- jvm
categories:
    - java
keywords: hotspot,classLoader,双亲委派,类加载器
description: 什么是JAVA类加载？JAVA类什么时候类加载？类加载都有哪些过程？类加载机制是怎么样的？双亲委派模型设计特点是怎么样的？有哪些优点？OSGI的类加载器为什么会火？OSGI类加载器原理是怎么样的？来来来一起说道说道。
---


## 什么是类加载 ##

类的加载指的是将类的class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且提供了访问方法区内的数据结构的接口。

## <span id = "jump">类加载的时机</span> ##

什么情况下需要开发类的加载过程，java虚拟机规范中并没有进行强制约束，但是对于初始化阶段，虚拟机规范有严格的规定：

- 遇到new、getstatic、putstatic或者invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。
- 使用java.lang.reflect包的方法对类进行发射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类，虚拟机会先初始化这个主类。
- 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MrthodHandle实例最后的解析结果是REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄时，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

> 注：当一个类在初始化时，要求其父类全部都已经初始化过了，但是一个接口在初始化时，并不需要其父接口全部完成了初始化，只有在真正使用到父接口的时候才会初始化。

## 类加载的过程 ##

![类加载的过程](https://github.com/gamesdoa/img0/raw/master/java/hotspot-classloader/class%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.jpg)

类的生命周期包括：加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)、卸载(Unloading) 7个阶段，其中验证、准备、解析3个部分统称为连接(Linking)。

类加载的过程包括了<font color=#FF0000>加载、验证、准备、解析、初始化</font>五个阶段。在这五个阶段中，**加载、验证、准备和初始化**这四个阶段发生的顺序是确定的，而**解析阶段**则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定（也成为动态绑定或晚期绑定）。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

### 加载 ###
加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下3件事情：

- 通过一个类的全限定名来获取定义此类的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

> 注：相对于类加载过程的其他阶段，一个非数组类的加载阶段(准确的说，是类加载阶段中获取类的二进制字节流的动作)是开发人员可控性最强的，因为加载阶段既可以使用系统提供的引导类加载器来完成，也可以由用户自定义的类加载器去完成，开发人员可以通过定义自己的类加载器去控制字节流的获取方式（重写类加载器的loadClass()方法）。
> 
> 加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，然后在内存中实例化一个java.lang.Class类的对象，针对hotspot虚拟机而言，Class对象比较特殊，虽然是对象但是存放在方法区里面，这个对象将作为程序访问方法区中的这些类型数据的外部入口。

### 连接 ###

#### 验证 ####
验证是连接阶段的第一步，目的：确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

从整体上看，验证阶段大致需要完成以下4个阶段的检验动作：

- 文件格式验证
> 验证字节流是否符合Class文件格式的规范，保证输入的字节流能正确的解析并存储于方法区之内，只有通过了这个阶段的验证后，字节流才会进入内存的方法区进行存储，后面3个验证阶段全部是基于方法区的存储结构进行的。

- 元数据验证
> 对字节码描述的信息进行语义分析，以保证其描述的信息符合java语义规范的要求

- 字节码验证
> 通过数据流和控制流分析，确定程序语义是合法的符合逻辑的，对类的方法进行校验分析，确保被验证类的方法在运行时不会做出危害虚拟机安全的事件。
 
- 符号引用验证
> 发生在虚拟机将符合引用转化成直接引用的时候，动作发生在连接的第三个阶段**解析**，验证对类本身以外(常量池中的各种符号引用)的信息进行匹配性校验。

	注：验证阶段对于虚拟机的类加载机制上一个非常重要但不是一定必要的阶段(对程序运行期没有影响)，
	对于进过反复验证的代码可以在启动的时候使用-Xverify:none参数关闭大部分的类验证，以缩短虚拟机类加载时间。

#### 准备 ####

正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。

- 内存分配只包括类变量(static)，而不包括实例变量，实例变量将会随着对象一起分配在java堆中
- 通常情况下数据类型赋值为零值，除非使用了static final标示一个变量，才会在这一阶段直接赋值。
> public static int value = 10;//准备阶段过后初始值为0，而不是10

> public static final int value = 10；//准备阶段过后初始值为10

#### 解析 ####
虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。

- 符号引用:符号可以是任何形式的字面量，与虚拟机内存布局无关，并且引用的目标并不一定已经加载到内存中。
 
- 直接引用:直接指向目标的指针，相对偏移量或者一个能间接定位到目标的句柄。与虚拟机内存布局相关，并且目标必然已经存在内存中。
### 初始化 ###
执行类构造器<clinit>()方法的过程，并为静态变量赋予正确的初始值，在Java中对类变量进行初始值设定有两种方式：

- 声明类变量是指定初始值
- 使用静态代码块为类变量指定初始值

> JVM初始化步骤
>
> 1. 假如这个类还没有被加载和连接，则程序先加载并连接该类
> 
> 2. 假如该类的直接父类还没有被初始化，则先初始化其直接父类
>
> 3. 假如类中有初始化语句，则系统依次执行这些初始化语句

类初始化时机：[见文第二部分,点击跳转](#jump)
## 类加载器 ##
定义：通过一个类的全限定名来获取描述此类的二进制字节流的动作放在虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类，这个动作的代码模块称为“类加载器”。

类唯一性：由加载它的类加载器和类本身一同才能确定唯一性，因为每个类加载器都拥有独立的类名称空间，所以比较两个类是否相等，需要由同一个类加载器加载为前提，否则两个类即使来源同一个class文件也必定不相等。
>注：相等包括Class对象的equals()、isAssignableFrom()方法、isInstance()方法的返回结果，还有instanceof关键字对象所属关系判定等情况。

### 类加载 ###
**类加载机制：**

>- 全盘负责:当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入

>- 父类委托:先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类([详情查看双亲委派模型](#pdm))

>- 缓存机制:缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效

**类加载有三种方式**：

 1. 命令行启动应用时候由JVM初始化加载
 2. 通过Class.forName()方法动态加载
 3. 通过ClassLoader.loadClass()方法动态加载

	```java
	package com.gamesdoa.classLoader;
	public class ClassLoaderTest {
	    public static void main(String[] args) throws ClassNotFoundException {
	        ClassLoader loader = ClassLoaderTest.class.getClassLoader();
	        System.out.println(loader);
	        //使用ClassLoader.loadClass()来加载类，不会执行初始化块
	        Class t = loader.loadClass("com.gamesdoa.classLoader.Test2");
	        //使用Class.forName()来加载类，默认会执行初始化块
	        //Class t = Class.forName("com.gamesdoa.classLoader.Test2");
	        //使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块
	        //Class t = Class.forName("com.gamesdoa.classLoader.Test2", false, loader);
	        System.out.println(t);
	    }
	}
	
	class Test2 {
	    static {
	        System.out.println("我被执行了！");
	    }
	}
	```

**Class.forName()和ClassLoader.loadClass()区别**

| Class.forName()        | ClassLoader.loadClass()            |
| ------------- |:-------------|
| 将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块      | 将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。 |


>注： Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 。

### *双亲委派模型 ###
![双亲委派模型](https://github.com/gamesdoa/img0/raw/master/java/hotspot-classloader/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%A8%A1%E5%9E%8B%E5%AE%8C%E6%95%B4%E7%89%88.jpg)

><font color=#FF0000>注:这里类加载器之间的父子关系不是以继承的关系来实现，而是使用组合关系来复用父加载器的代码</font>

	```java
	public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

	protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
			//首先，检查类是否已经被加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {//如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//父类不为null，也就是父类不是启动类加载器，委托给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {//父类为null，也就是父类是启动类加载器，检查是否满足启动类加载器限制，满足则委托给启动类加载器加载，否则返回null
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {//如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
	```

**<span id = "pdm">双亲委派的工作过程</span>**：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类，当最终的加载器加载失败时则会报出异常ClassNotFoundException。

**双亲委派模型目的**

- 系统类防止内存中出现多份同样的字节码

- 保证Java程序安全稳定运行

- 隔离不同加载器加载到的类对象

**自定义类加载**：当遇到类似以下情况的时候，需要自定义类加载：

1. 在执行非信任代码之前，自动验证数字签名。
2. 动态地创建符合用户特定需要的定制化构建类。
3. 从特定的场所取得java class，例如数据库中和网络，尤其是还需要对数据进行解密等操作的情况下。

>自定义类加载器一般都是继承自 ClassLoader 类，只需要重写 findClass 方法即可，如果不是因为特殊目的一般不要重写loadClass方法，因为这样容易破坏双亲委托模式。

	```java
	package com.gamesdoa.classLoader;
	
	import java.io.ByteArrayOutputStream;
	import java.io.File;
	import java.io.FileInputStream;
	import java.lang.reflect.InvocationTargetException;
	import java.lang.reflect.Method;
	
	public class MyClassLoader extends ClassLoader {
	    private String classpath;
	
	    public MyClassLoader(String classpath){
	        this.classpath = classpath;
	    }
	
	    @Override
	    public Class<?> findClass(String name){
	        byte[] data = loadClassData(name);
	        return this.defineClass(name , data , 0 , data.length);
	    }
	
	    public byte[] loadClassData(String name){
	        System.out.println("加载"+name);
	        try{
	            name = name.replace("." , "//");
	            FileInputStream is = new FileInputStream(new File(classpath + name + ".class"));
	            ByteArrayOutputStream baos = new ByteArrayOutputStream();
	            int b = 0 ;
	            while((b = is.read()) != -1){
	                baos.write(b);
	            }
	            is.close();
	            return baos.toByteArray();
	        } catch (Exception e){
	            e.printStackTrace();
	        }
	        return null;
	    }
	
	    public static void main(String[] args) throws InstantiationException, IllegalAccessException,
	            ClassNotFoundException, NoSuchMethodException, SecurityException, IllegalArgumentException,
	            InvocationTargetException {
	        MyClassLoader myLoader = new MyClassLoader("E:/");
	
	
	        Class<?> clazz = myLoader.loadClass("com.gamesdoa.classLoader.Apple");
	        Class<?> clazz1 = myLoader.loadClass("com.gamesdoa.classLoader.Apple");
	        System.out.println(clazz == clazz1);
	        System.out.println(clazz.getClassLoader());
	        System.out.println(clazz1.getClassLoader());
	
	        System.out.println("================");
	
	        MyClassLoader myLoader1 = new MyClassLoader("E:/");
	        Class<?> clazz2 = myLoader1.loadClass("com.gamesdoa.classLoader.Apple");
	        System.out.println(clazz2 == clazz1 );
	        System.out.println(clazz2.getClassLoader());
	
	        System.out.println("================");
	
	        Object newInstance = clazz.newInstance();
	        Method declaredMethod = clazz.getDeclaredMethod("say", new Class[]{});
	        declaredMethod.invoke(newInstance, new Object[]{});
	    }
	}
	
	class Apple {
	    static{
	        System.out.println("int Apple");
	    }
	
	    public void say(){
	        System.out.println("I am a apple..");
	    }
	}

	```
>这个类本身可以被 AppClassLoader 类加载，因此我们不能把 com/gamesdoa/classLoader/MyClassLoader.class 放在类路径下。否则，由于双亲委托机制的存在，会直接导致该类由 AppClassLoader 加载，而不会通过我们自定义类加载器来加载。

### 破坏双亲委派模型(OSGI) ###
双亲委派模型出现过3次大规模的“被破坏”情况

- 双亲委派模式出现之前：JDK1.2之前
- 线程上下文类加载器(Thread Context ClassLoader): 基础类又要调用回用户的代码(JNDI、JDBC、JCE、JAXB、JBI等)
- 用户对于程序动态性的追求:代码热替换(HotSwap)、模块热部署(Hot Deploment)等

**OSGI(Open Service Gateway Initiative)**：java模块化标准：每一个程序模块(OSGI中称Bundle)都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。因此在OSGI环境中类加载器发展为复杂的网状结构。下面来分析一下这个类加载器模型。

**OSGI各个Bundle类加载之间的规则：**

- 某个Bundle声明了一个它依赖的Package，如果有其他Bundle声明发布了这个Package，那么所有对于这个Package的类加载动作都会委派给发布它的Bundle类加载器去完成。
- 不涉及某个具体Package时，各个Bundle加载器都是平级关系，只有具体使用某个Package和Class的时候，才会根据Package导入导出定义来构造Bundle间的委派和依赖。
- 一个Bundle类加载器为其他Bundle提供服务时，会根据Export-Package列表严格控制访问范围。一个类存在于Bundle的类库但是没有被Export，那么这个Bundle的类加载器能找到这个类，但是不会提供给其他Bundle使用，而且OSGI平台也不会把其他Bundle的类加载请求分配给这个Bundle来处理。

**OSGI类加载器架构：**

![OSGI类加载器架构](https://github.com/gamesdoa/img0/raw/master/java/hotspot-classloader/OSGI%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E6%9E%B6%E6%9E%84.jpg)

**OSGI类加载时查找规则如下：**

- 以java.* 开头的类，委派给父类加载器加载。
- 否则，委派列表名单内的类，委派给父类加载器加载。
- 否则，Import 列表中的类，委派给Export这个类的Bundle的类加载器加载。
- 否则，查找当前Bundle 的 Classpath，使用自己的类加载器加载。
- 否则，查找是否在自己的 Fragement Bundle 中，如果是，则委派给 Fragment Bundle 的类加载器加载
- 否则，查找Dynamic Import 列表的 Bundle， 委派给对应Bundle的类加载器加载。
- 否则，查找失败。

>注：OSGI的Bundle相互引用有可能会造成死锁，因为当前类加载器在实例对象时需要先锁定，java.lang.ClassLoader.loadClass()方法是一整个synchronized块。当BundleA依赖PackageB,同时BundleB依赖PackageA时可能会死锁。
>
>loadClass的这个特性也就造就了**饿汉模式**的**单例模式实现**