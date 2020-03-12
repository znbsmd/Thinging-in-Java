---
title: Thinking in Java 第二十一章
date: 2020-03-03 10:24:28
tags:
categories: Java
---
# 第二十一章 并发

目前为止，你学到的都是顺序编程知识，即程序重的所有事物在任意时刻都只能执行一个步骤。

## 21.1 并发的多面性

用并发解决问题大体上分为：“速度”和“设计可管理性”两种。

#### 21.1.1 更快的执行

多处理器可以将大量用户的需求分不到多个CPU中，但是并发通常是提高运行在单处理器上的程序。在**单处理器上运行的并发程序开销确实应该比大部分顺序执行开销大**。因为增加了上下文切换的代价。**如果没有阻塞，那么在单处理机器上使用并发就没有任何意义。**
在单处理系统中的性能提高的常见示例是事件驱动编程。通过创建单独的执行线程来响应用户的输入，即使这个线程在大多数时间里都是阻塞的，但是程序可以保证具有一定程度的可响应性。
java 不支持多任务，Java 添加多线程机制。

#### 21.1.2 改进代码设计

- 在Java中，通常要假定你不会获得足够的线程，从而使得可以为大型仿真中的每个元素都提供一个线程。
- 解决这个问题的典型方式是使用协作多线程。Java的线程机制是抢占式的，这表示调度机制会周期性地中断线程，将上下文切换到另一个线程，从而为每个线程都提供时间片，使得每个线程都会分配到数量合理的时间去驱动它的任务。
- 协作式系统的优势：上下文切换的开销通常比抢占式系统要低廉许多，并且对可以同时执行的线程数量在理论上没有任何限制。

## 21.2 基本的线程机制

使用线程机制是一种建立透明的、可扩展的程序方法，如果程序运行太慢，为机器添加cpu。多任务和多线程往往是多处理器系统的最合理方式。

#### 21.2.1 定义任务

```
package Thread;

public class LiftOff implements Runnable {

    protected int CountDown = 10;
    private static int taskCount = 0;
    private final int id = taskCount++;

    public LiftOff(){}
    public LiftOff(int countDown) {
        CountDown = countDown;
    }

    public String status(){
        return "#" + id + "(" + (CountDown > 0 ? CountDown : "off") + "), ";
    }
    @Override
    public void run() {
        while (CountDown-- >0){

            System.out.println(status());
            Thread.yield();// 线程调度器，一个线程转移到另一个线程
        }
    }
}
```

```
package Thread;

public class MainThread {
    public static void main(String[] args) {
        LiftOff liftOff = new LiftOff();
        liftOff.run();
    }
}

```
#### 21.2.2 Thread 类

Thread构造器只需要一个Runnable对象。
```
Thread thread = new Thread(new LiftOff());
thread.start();
```

**main()方法也是一个线程，所以main()和LiftOff()是同时执行的。**
线程调度机制是非确定性的，不可以依赖线程的代码，尽可能的保守使用线程。
线程虽然没有捕获对象的引用，但是注册了自己，所以start()之后仍然不会被垃圾回收。
**thread的run方法还在执行时候不管这个thread对象处在什么状态下，thread对象都不会被回收，因为Thread的run()方法的局部变量this保持了对线程对象Thread的引用.**
所以start()仅仅是线程的开始。


#### 21.2.3 使用Executor

Java SE5的java.util.concurrent中的执行器（Executor）将为你管理Thread对象，从而简化了并发编程。 （线程池）
```
ExecutorService exec = Executors.newFixedThreadPool(10);
for (int i = 0; i < 5; i++) {
    exec.execute(new LiftOff());
}
exec.shutdown();
```

- FixedThreadPool使用了固定的线程集来执行所提交的任务。
- CachedThreadPool在程序执行过程中通常会创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程，因此它是合理的Executor的首选。
- SingleThreadExecutor 顺序执行 只会创建一个线程执行 单例线程。

#### 21.2.4 从任务中产生返回值

实现Callable接口 可从任务中产生返回值，submit()会产生Future对象，用Future 对象接收返回参数。

```
package Thread;

import java.util.concurrent.Callable;

public class TaskWithResult implements Callable {
    private int id;

    public TaskWithResult(int id) {
        this.id = id;
    }


    @Override
    public Object call() throws Exception {
        System.out.println("asd");
        return "result of TaskWithResult" + id;
    }
}

```

```
package Thread;

import java.util.ArrayList;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableDemo {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();

        ArrayList<Future<String>> futures = new ArrayList<>();

        Future s = executorService.submit(new TaskWithResult(2));

        try {
            System.out.println(s.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        s.isCancelled();
//        for (int i = 0; i < 10; i++) {
//            futures.add(executorService.submit(new TaskWithResult(i)));
//        }
//
//        for (Future<String> fs: futures){
//            try {
//                System.out.println(fs.get());
//            } catch (InterruptedException e){
//                System.out.println(e);
//            } catch (ExecutionException e) {
//                e.printStackTrace();
//            } finally {
//                executorService.shutdown();
//            }
//        }
    }
}

```

**execute提交的方式只能提交一个Runnable的对象，且该方法的返回值是void，也即是提交后如果线程运行后，和主线程就脱离了关系了。 submit 接收 Callable 接口的实现类，可用get()接收返回结果**

#### 21.2.5 休眠
Thread.sleep(1000);

#### 21.2.6 优先级

绝大时间里，都应该以默认优先级运行。试图操作通常是一种错误。
Thread.currentThread().setPriority(Thread.MAX_PRIORITY);
**优先级在run()开头设置，不要在构造器里设定，因为Executor 还没有开始执行任务**
只有执行了100000次开销相当大的浮点运算，包括double类型的加法与除法。没有这些运算，看不到优先级的效果。

#### 21.2.7 让步

如果知道已完成了在run()方法所需的工作，就可以给线程调度一个暗示，告诉它做的差不多了，可以让别的线程使用cpu，这个暗示用yield()方法，不过不能保证他会被采纳。当调用yield时，你也是在建议有相同优先级的其他线程可运行。
（yield并不意味着退出和暂停，只是，告诉线程调度如果有人需要，可以先拿去，我过会再执行，没人需要，我继续执行）调用yield的时候锁并没有被释放。对于任何重要的控制或在调整应用时，都不恩那个依赖于yield。实际上，yield经常被误用。
- **解读：不要用yield控制程序，它只是调度线程让步。**

#### 21.2.8 后台程序

必须在线程启动之前start()前调用setDaemon()方法，才能把它设置为后台线程。
当最后一个非后台线程终止时，后台线程会“突然”停止。
isDaemon()来确定线程是否是一个后台线程。如果是 ，它创建的任何线程就是一个后台线程。
- **守护线程不能单独存活在JVM，即当程序中用户线程全部结束后JVM会立即杀死所有守护线程，守护线程中存在finally代码块，那么当所有的非守护线程中止时，守护线程被kill掉，其finally代码块是不会执行的。(除非main（）还在)，所以一般不要使用后台线程，因为你不能优雅的关闭它，非后台的Executor 通常是一种更好的方式。**

#### 21.2.9 编码的变体
可直接继承Thread类，但是没办法继承其他类，所以最灵活的还是实现Runnable。

#### 21.2.10 术语

描述要执行的工作时使用术语 “任务”，只有引用到驱动任务的具体机制时候，才使用“线程”。
实现Runnable只是定义一个“任务”，继承Thread 也是 继承出一个任务。在Java中，Thread类自身不执行任何操作，他只是驱动赋予他的任务。**Thread类并没有任何控制权，并且在隔离高的执行器（在执行器里创建任和管理）更加明显。**

#### 21.2.11 加入一个线程


- join() 定义在Thread.java中。
- join() 的作用：让“主线程”等待“子线程”结束之后才能继续运行。

```
// 主线程
public class Father extends Thread {
    public void run() {
        Son s = new Son();
        s.start();
        s.join();
        ...
    }
}
// 子线程
public class Son extends Thread {
    public void run() {
        ...
    }
}
```
在调用s.join()之后，Father主线程会一直等待，直到“子线程s”运行完毕；在“子线程s”运行完毕之后，Father主线程才能接着运行。 这也就是我们所说的“join()的作用，是让主线程会等待子线程结束之后才能继续运行”！

**wait()的作用是让“当前线程”等待**，而这里的“当前线程”是指当前在CPU上运行的线程。所以，虽然是调用子线程的wait()方法，但是它是通过“主线程”去调用的；所以，休眠的是主线程，而不是“子线程”！

join()源码中 isAlive()检查此线程是否存活。

以上引用 https://www.cnblogs.com/skywang12345/p/3479275.html

对join()方法的调用可以被中断，做法是在调用线程上调用interrupt()方法。
**注：谁加入 谁调用interrupt**

#### 21.2.12 创建有响应的用户界面

使用线程的动机之一就是建立有响应的用户界面. 利用设置守护进程 setDaemon() 可以在等待用户输入的同时，后台操作浮点运算。 

#### 21.2.13 线程组

线程组持有一个线程集合。 ThreadGroup（最好把线程组堪称一次不成功的尝试，直接忽略！
！！）

#### 21.2.14 捕获异常

当单线程的程序发生一个未捕获的异常时我们可以采用try....catch进行异常的捕获，但是在多线程环境中，线程抛出的异常是不能用try....catch捕获的。
Thread.UncaughtExceptionHandler是Java SE5中的接口，允许在每个Thread对象上附着一个异常处理器。

使用示例：
```
package com.exception;
 
import java.lang.Thread.UncaughtExceptionHandler;
 
public class WitchCaughtThread
{
	public static void main(String args[])
	{
		Thread thread = new Thread(new Task());
		thread.setUncaughtExceptionHandler(new ExceptionHandler());
		thread.start();
	}
}
 
class ExceptionHandler implements UncaughtExceptionHandler
{
	@Override
	public void uncaughtException(Thread t, Throwable e)
	{
		System.out.println("==Exception: "+e.getMessage());
	}
}
```

## 21.3 共享受限资源

#### 21.3.1 不正确地访问资源

java 递增也不是原子性的，自身也需要多个步骤，因此不保护任务，单一递增也不是安全的。

#### 21.3.2 解决共享资源竞争

基本上所有的并发模式在解决线程冲突问题的时候，都是采用序列化访问共享资源的方案。这意味着在给定时刻只允许一个任务访问共享资源。

在使用并发时，将域设置为private是非常重要的。

使用显式的Lock对象

Lock对象必须被显式地创建、锁定和释放。

有了显式的Lock对象，就可以使用finnally子句将系统维护在正确的状态。

**Lock相比于synchronized具有更细粒度的控制力，以及处理异常，比如遍历列表中的节点的加锁机制（锁耦合），必须在当前节点解锁后捕获下一个锁。**


#### 21.3.3 原子性与易变性

原子操作是不能被线程调度机制中端的操作；一旦操作开始，那么它一定可以再发生的“上下文切换”之前（切换到其它线程执行）执行完毕。

**但是还是一种过于简化的机制，实际上也可能不安全，因为在多处理器上 还有可视性问题，例如一个任务作出修改，对其他任务也可能是不可视的，（修改只是暂时性存储本地处理器的缓存中），如果没有同步机制，那么修改时的可视将无法确定。**

如果一个域完全由synchronized方法或语句块来防护，那就不必将其设置为volatile。使用volatile而不是synchronized的唯一安全情况是类中只有一个可变的域。

**对基本类型的读取和赋值以及递增操作不是安全的原子性操作。反编译后将看到很多指令，get put 等等。**

#### 21.3.4 原子类

Atomic类被设计用来构建java.util.concurrent中的类，因此只有在特殊情况下才使用它们。如果涉及多个Atomic对象，可能就应该考虑放弃使用Atomic，通常依赖于锁要更安全一点。（要么是synchronized关键字，要么是显示的Lock对象）

#### 21.3.5 临界区

同步控制块 例如：
```
synchronized(syncObject) {
    // 临界区
}
```
#### 21.3.6  在其它对象上同步

```
package synchronize;

public class DualSynch {

    Object o = new Object();
    public synchronized void f(){
        for (int i = 0; i < 5; i++) {

            System.out.println("f()");
            Thread.yield();
        }
    }

    public void g(){
        synchronized (o){

            for (int i = 0; i < 5; i++) {

                System.out.println("g()");
                Thread.yield();
            }
        }
    }


}

```
```
package synchronize;

public class SynObject {
    public static void main(String[] args) {
        DualSynch dualSynch = new DualSynch();

        new Thread(){
            @Override
            public void run() {
                dualSynch.f();
            }
        }.start();

        dualSynch.g();
    }
}

```
一个简答的例子验证了   synchronized 锁的内容。如果在this上同步，临界区的效果就会直接缩小在同步的范围内。两个任务可以同时进入同一个对象，只要这个对象上的方法是在不同的锁上同步的即可。
**synchronized代码块本质上完成的是代码片段的自动上锁和解锁，以确保关键代码片段在多线程中的互斥访问。synchronized既可以加在一段代码上，也可以加在方法上。**
（幼儿园解析）
synchronized 有三种形式 ：
1. 静态方法上的锁

    静态方法是属于“类”，不属于某个实例，是所有对象实例所共享的方法。也就是说如果在静态方法上加入synchronized，那么它获取的就是这个类的锁，**锁住的就是这个类。**
2. 实例方法（普通方法）上的锁
 
    实例方法并不是类所独有的，每个对象实例独立拥有它，它并不被对象实例所共享。这也比较能推出，在实例方法上加入synchronized，那么它获取的就是这个类的锁，**锁住的就是这个对象实例。**
3.  方法中使用同步代码块

    - synchronized(this){...} this关键字所代表的意思是该对象实例，换句话说，这种用法synchronized锁住的仍然是对象实例，他和public synchronized void demo(){}可以说仅仅是做了语法上的改变。
    - private Object obj = new Object();    synchronized(obj){...}
    - synchronized(Demo.class){...}
    这种形式等同于抢占获取类锁.收效甚微

以上引用：https://juejin.im/post/5d0c8101e51d455a694f954a

#### 21.3.7 线程本地存储

防止共享资源发生冲突的第二种方式就是根除对变量的共享。-> java.lang.ThreadLocal 实现。

ThreadLocal 通常当作静态域存储，创建ThreadLocal时，只能通过get() set() 访问对象的内容。

原理解读：ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象。

**所以简单说底层就是往Map存值，一个key 就是一个线程，就不会造成资源冲突了。**

## 21.4 终结任务

#### 21.4.1 装饰性花园

本小节用了一大段代码说明了 多线程从统计学角度说失败的概率很小，导致很容易掉进坑里。所以要验证 要常用yield()，去增加失败率。可以用awaitTermination()方法来判断线程池中是否有继续运行的线程。

####21.4.2 在阻塞时终结

线程状态

1. 新建（new）
2. 就绪（runnable）只要调度器把时间片分给线程就会执行。
3. 阻塞（blocked）某个条件会阻止运行。当进入阻塞，调度器不会分配时间给线程，直到恢复就绪状态。
4. 死亡（dead）结束或中断。

进入阻塞状态

1. 通过调用sleep(milliseconds)使任务进入休眠状态。
2. 调用wait()使线程挂起。直到线程得到了notify()或notifyAll()消息。
3. 任务在等待某个输入/输出完成。（synchronized）
4. 试图在某个对象上调用其同步控制方法，但对象锁不可用，因为另一个任务已经获得了这个锁。

####21.4.3 中断

Thread interrupt()方法中止阻塞重的线程会抛出 InterruptedException() 异常。

concurrent类库避免对Thread对象的直接操作，转而通过Executor 执行所有操作。如果在Executor上调用shutdownNow()，它将发送一个interrupt()调用给它启动的所有线程。

使用Executor，那么通过调用submit()而不是execute()来启动任务，就可以持有该任务的上下文。submit()将返回一个泛型Future<?>，其中有一个未修饰的参数，因为你永远都不会在其上调用get()——持有这种Future的关键在于你可以在其上调用cancel()，并因此可以使用它来中断某个特定任务。如果你将true传递给cancel()，那么它就会拥有在该线程上调用interrupt()以停止这个线程的权限。因此，cancel()是一种中断由Executor启动的单个线程的方式。

**shutdown() 内部正在跑的任务和队列里等待的任务，会执行完。shutdownNow() 尝试将正在跑的任务interrupt中断.忽略队列里等待的任务,返回未执行的任务列表。**

不能中断正在试图获取synchronized锁或者试图执行I/O操作的线程。有一个不太好的解决方案：关闭任务在其上发生阻塞的底层资源。
例如 xx.close(); 关闭流资源。
（扩展基础）synchronized 释放 条件有4个：
    1. 当前线程的同步方法、代码块执行结束的时候释放。
    2. 当前线程在同步方法、同步代码块中遇到break 、 return 终于该代码块或者方法的时候释放。
    3. 出现未处理的error或者exception导致异常结束的时候释放
    4. 程序执行了 同步对象 wait 方法 ，当前线程暂停，释放锁

各种nio类提供了更人性化的I/O中断，被阻塞的nio通道会自动地响应中断。-> SocketChannel.close();

**被互斥的阻塞**

一个任务能够调用在同一个对象中的其他的synchronized方法，因为这个任务已经持有锁了。

Java SE5并发类库添加了一个特性，即在ReentrantLock上阻塞的任务具备可以被中断的能力。

#### 21.4.4 检查中断

Thread.interrupted()检查中断状态，不仅可以告诉你interrupt()是否被调用过，也可以清除中断状态。

try-finally子句使得无论run()循环如何退出，清理都会发生.

## 21.5 线程之间的协作

当一个任务在方法里遇到了对wait()的调用的时候，线程的执行被挂起，**对象上的锁被释放**。
wait()、notify()和notifyAll()是基类Object的一部分。实际上，只能在同步控制方法或同步控制块里调用wait()、notify()和notifyAll()。

**错失的信号**

```
thread1:
synchronized(sharedMonitor)
{
      someCondition = false;
      sharedMonitor.notify();
}
 
thread2:
while(someCondition)
{
	//point 1
	synchronized(sharedMonitor)
	{
		sharedMonitor.wait();
	}
}

```

先执行thread2，当进入while循环 执行thread1 会出现无限期等待。所以要把thread2代码块都同步。防止在在someCondition变量上产生竞争条件。

#### 21.5.2 notify()与notifyAll()

在技术上，可能会有多个任务在单个Car对象上处于wait（）状态，notifyAll()更加安全。蛋实际上只会有一个任务处于wait(),因此用notify()替代notifyAll()。

**当前的线程不是此对象监视器的所有者。也就是要在当前线程锁定对象，才能用锁定的对象此行这些方法，需要用到synchronized ，锁定什么对象就用什么对象来执行  notify(), notifyAll(),wait(), wait(long), wait(long, int)操作，否则就会报IllegalMonitorStateException异常。**

**替代notifyAll() 将唤醒正在等待的所有任务是不严谨的，只是对当前线程锁定对象，等待这个锁的任务才会被唤醒。**

使用notify()，就必须保证被唤醒的是恰当的任务。

#### 21.5.3 生产者与消费者

生产者-消费者实现中，应使用先进先出队列来存储被生产和消费的对象。

**使用显式的Lock和Condition对象**

需要条件判断。而单个Lock将产生一个Condition对象，这个对象被用来管理任务间的通信。其他仍需要额外的表示处理状态信息。所以一个类中多个条件锁等待，需要用Condition()。

#### 21.5.4 生产者-消费者队列

wait() notifyAll() 以一种非常低级的方式解决了任务互操作问题，即每次交互时候都握手。多数情况下，可以使用更高级抽象级别，同步队列->(BlockingQueue，其内部是同步的）和系统的设计隐式地管理了任何时刻都只有一个任务在操作。**但是遇到多消费者按顺序消费的需求用实现类LinkedBlockingQueue仍然需要外部加同步，它只是保证不会超消费，并不能保证多线程抢占顺序，还可以使用实现类SynchronousQueue。**


#### 21.5.5 任务间使用管道进行输入/输出

PipedWriter/PipedReader 管道流通信核心是,Writer和Reader公用一块缓冲区,缓冲区在Reader中申请,由Writer调用和它绑定的Reader的Receive方法进行写. （BlockingQueue 使用起来会更加好。）

## 21.6 死锁

任务之间相互等待的连续循环，没有哪个线程能继续。被称之为死锁。最可怕的是看起来工作良好，但是具有死锁危险。（很难重现的方式发生。）

死锁的条件：

- 互斥条件
- 至少有一个任务必须持有一个资源且正在等待获取一个当前被别的任务持有的资源
- 资源不能被任务抢占，任务必须把资源释放当做普通事件
- 必须有循环等待

破除一条就会解除，最容易的方法就是破坏第四个条件。

## 21.7 新类库中的构件

#### 21.7.1 CountDownLatch

闭锁。被用来同步一个或多个任务，强制它们等待由其它任务执行的一组操作完成。
countDown 计数达到0, await() 调用会被阻塞。

#### 21.7.1 CyclicBarrier

栅栏。CyclicBarrier用于等待其他线程运行到栅栏位置。与CountDownLatch 相似，CountDownLatch是只触发一次的事件，而CyclicBarrier可以多次重用。

#### 21.7.3 DelayQueue

延时队列。无届的BlockingQueue，放置的对象实现了Delay接口对象，其中的对象只能在其到期时才能从队列取走。

#### 21.7.4 PriorityBlockingQueue

优先级队列PriorityBlockingQueue必须是实现Comparable接口，队列通过这个接口的compare方法确定对象的priority。当前和其他对象比较，如果compare方法返回负数，那么在队列里面的优先级就比较高。

#### 21.7.5 使用ScheduleExecutor的温室控制器

ScheduledThreadPoolExecutor 能解决在预定时间运行的任务。（线程池）

#### 21.7.6 Semaphore

正常的锁只能允许一个任务访问同一个资源。Semaphore可以控制某个资源可被同时访问的个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。比如在Windows下可以设置共享文件的最大客户端访问个数。 

#### 21.7.7 Exchanger

应用场景是：一个任务在创建对象，这些对象的生产代价很高昂，而另一个任务在消费这些对象。通过这种方式，可以有更多的对象在被创建的同时被消费。**创建完直接给其他线程处理消费，无需等待线程去选择执行。**

## 21.8 仿真

个人理解：多线程可以模拟出真实应用场景的并发需求。

## 21.9 性能调优

用单线程测试 synchronized 要比 Lock 快很多,但是并没有意义。有可能会执行特殊优化，并且固定次数，可能会被编译器识别提前计算。所以要在多线程环境下测试。

#### 21.9.1 比较各类互斥技术

Lock通常会比使用synchronized要高效许多并且稳定。

**1.6后synchronized优化为锁升级，效率相近。**

#### 21.9.2 免锁容器

对容器的修改可以与读取操作同时发生，只要读取者只能看到完成修改的结果即可。

CopyOnWriteArrayList/CopyOnWriteSet/ConcurrentHashMap 都可称为免锁容器

**只有set使用synchronized，原理是在容器数据结构复制出一个单独的副本，修改结束后与主数据交换。使新读操作可以看到这个修改。容器只有部分内容而不是整个内容可以被复制和修改。**

源码如下：
```
public boolean add(E e) {
        synchronized (lock) {
            Object[] es = getArray();
            int len = es.length;
            es = Arrays.copyOf(es, len + 1);
            es[len] = e;
            setArray(es);
            return true;
        }
    }
```
等到修改结束，读操作才能看到。

**乐观锁**

synchronized ArrayList 无论读写数量多少，性能都大致相同的慢。CopyOnWriteArrayList对写少的情况喜爱回异常的快，并且在多写情况下虽然慢但也要比 整个列表加锁性能要好的多。

**比较各种Map实现  **

ConcurrentHashMap 比synchronized HashMap 性能要好很多，因为使用分段锁，每次只锁一个Entry的key值。

#### 21.9.3 乐观加锁
Atomic对象的compareAndSet() 比较并交换 不需要互斥锁，性能更高，但是比较失败 需要重续操作，比较消耗性能
someone.compareAndSet(false,true)可以理解为：

```
if(someone == false){
    someone=true
}
```

#### 21.9.4 ReadWriteLock

对向数据结构相对不频繁的写入，但是有多个任务要经常读取这个数据结构的这类情况进行了优化。

## 21.10 活动对象
活动对象的特征：
 - 基于消息机制：
 对活动对象的请求和可能的参数都被转化为消息，这些消息被转发给活动对象实际实现并排队等待处理。处理结果以future对象返还给提出请求的对象。

- 异步调用：
对活动对象的请求被异步执行，实际由活动对象的工作线程处理请求，故不会阻塞调用者。仅当请求未完成执行时，调用者试图获取处理结果时会被阻塞。

- 线程安全：
活动对象天生是线程安全的。因为他顺序地从请求队列摘取消息进行处理，并且始终在一个单独的线程中执行，由于不存在资源竞争，所以也不用担心同步、死锁等问题。同步仍旧会发生，但它通过将方法调用排队，使得任何时刻都只能发生一个调用，从而将同步控制在消息的级别上发生。
每个Future对象 在维护着它自己的工作器线程和消息队列，并且所有对这种对象的请求都将进入队列排队，任何时刻只运行一个。

应用场景：
适合于按某一特定顺序执行而又互不影响的调用者执行状况的情景。

## 21.11 总结

 线程提供轻量级执行上下文切换（大约100条指令。）
 进程是重量的（大约1000条。）
 线程缺点：
 1. 等待共享资源性能降低
 2. 需要处理线程的额外CPU花费。
 3. 糟糕的程序设计导致不必要饿复杂度。
 4. 有可能产生一些病态行为，如饿死、竞争、死锁、活锁（多个运行各自任务的线程使得整体无法完成。）
 5. 不同平台导致不一致性。

 多线程共享资源，要确保不会同时读取或改变资源，明知的使用加锁机制。要对锁有透彻的理解，否则会引入潜在的死锁。

 如果线程问题变得大而复杂，就要使用erlang语言。专门用于线程机制的函数型语言。
