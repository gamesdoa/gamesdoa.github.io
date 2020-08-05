---
title: Thread中start和run方法的区别
date: 2017-09-28
tags: 
	- java
	- Thread
categories:
    - java
keywords: spring Thread 多线程
description: 工作使用java多年，没有分清java中的线程在run和start方法的的区别，通过API和一个例子来看一下到底他们有什么区别
---


# 认识Thread的 start() 和 run() #
我们通过API定义来看一下start() 和 run()的区别
## start() ##
JDK中的注释是这样的：

	Causes this thread to begin execution; 
	the Java Virtual Machine calls the run method of this thread.
    The result is that two threads are running concurrently: 
	the current thread (which returns from the call to the start method) 
	and the other thread (which executes its run method).
    It is never legal to start a thread more than once.
    In particular, a thread may not be restarted once it has completed execution.

大体意思是：可以是线程开始执行，java虚拟机将会调用次线程的run方法。这样运行的结果是将会有两个线程同时运行（一个是当前线程，也就是调用start方法的线程，另外一个是执行run方法的线程），并且该方法启动线程只能启动一次，不允许重复启动，也就是说一旦线程完成执行，线程就不能重新启动。

> 用start方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码。通过调用Thread类的 start()方法来启动一个线程，这时此线程处于就绪（可运行）状态，并没有运行，一旦得到cpu时间片，就开始执行run()方法，这里方法 run()称为线程体，它包含了要执行的这个线程的内容，Run方法运行结束，此线程随即终止。


## run() ##
JDK中的注释是这样的：

> When an object implementing interface  Runnable is used to create a thread, starting the thread causes the object's run method to be called in that separately executing  thread.

> The general contract of the method run is that it may  take any action whatsoever.

大体意思是：使用实现接口Runnable的对象来创建线程时，启动时会在单独执行的线程中调用run方法。一般来说，该方法不执行任何的操作返回（void）。

> Thread 的子类应该重写该方法。

>run()方法只是类的一个普通方法，如果直接调用Run方法，程序中依然只有主线程这一个线程，其程序执行路径还是顺序执行，要等待run方法体执行完毕后才可继续执行下面的代码，这样就没有达到写线程的目的。

## 总结对比 ##

从API不难看出，start方法是启动一个线程并会调用内部的run方法运行，而run方法只是thread的一个普通方法，他的执行还是在当前线程中。

# 一个例子 #

	package com.gamesdoa.test.thread;

	import org.junit.Test;
	import java.util.concurrent.TimeUnit;
	/**
	 * Created by gamesdoa on 2017/9/28.
	 */
	public class TreadRandT {
	    @Test
	    public void testStart() throws InterruptedException {
	        Thread t = new Thread() {
	            public void run() {
	                call();
	            }
	        };
	        t.start();
			try {
	            t.start();
	        }catch (Exception e){
	            e.printStackTrace();
	        }
        	System.out.println("线程：" + Thread.currentThread().getName() 
                + "; 我是主线程");
	        TimeUnit.MILLISECONDS.sleep(100l);
	    }
	
	    @Test
	    public void testRun() {
	        Thread t = new Thread() {
	            public void run() {
	                call();
	            }
	        };
	        t.run();
	        t.run();
        	System.out.println("线程：" + Thread.currentThread().getName()
                + "; 我是主线程");
	    }
	
	    private void call() {
        	System.out.println("线程：" + Thread.currentThread().getName()
                + "; call被调用了");
	    }
	
	    @Test
	    public void testAll() throws InterruptedException {
	        testStart();
	        System.out.println("------------我是华丽的分割线--------------");
	        testRun();
	    }
	}

运行结果如下：

	调用start方法：
	线程：main; 我是主线程
	java.lang.IllegalThreadStateException
		at java.lang.Thread.start(Thread.java:705)
		at com.gamesdoa.test.thread.TreadRandT.testStart(TreadRandT.java:21)
		at com.gamesdoa.test.thread.TreadRandT.testAll(TreadRandT.java:51)<27 internal calls>
	线程：Thread-0; call被调用了
	
	------------我是华丽的分割线--------------
	
	调用run方法：
	线程：main; call被调用了
	线程：main; call被调用了
	线程：main; 我是主线程

>注 ： 可以看到，我们在调用 start 方法启动线程的时候会有一个新的线程被启动（Thread-0），在Thread-0里面才执行的call调用，是一个单独的线程，这样也就可以处理成多线程系统。 重复调用start将会报IllegalThreadStateException异常。
>
> 再看 run 方法，该方法的调用call执行的时候，还是在主线程中顺序执行，调用 run 方法在前，也就先执行， 主线程的输出在后，也就后执行，同样这里可以看到重复执行run方法时完全允许的。

> 为什么会在start的方法执行完之后休眠100毫秒？ 
> 
> 因为如果不加入休眠的话，当另外一个线程启动之后处于可运行状态，如果没有获取到CPU时间，主线程就直接运行完了，那么这里讲不会看到被调用的输出，这也正说明了多线程的执行规则，无序的。

# 线程状态 #
上面说到线程的状态，那么我们最后再在这里补充一下线程的状态：
![线程运行状态](https://github.com/gamesdoa/img0/raw/master/java/thread/thread-state.jpg)

> 上图可以看出来，线程有三个状态， 可运行 ， 运行中 ， 阻塞 。
> 
> 可运行在获取到CPU时间之后转换成运行中的状态
> 
> 运行中在CPU时间到期可以转换成可运行状态，同时遇到阻塞事件的时候状态就会转换成阻塞的状态
> 
> 阻塞状态在阻塞条件满足的情况下会变成可运行状态。