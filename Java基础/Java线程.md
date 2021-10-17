## 线程

### 创建线程的方式

* 继承Thread类，重写run方法。
* 实现Runnable接口，实现run方法。
* 使用Callable和Future创建线程。主要就是让线程有返回值，线程也可以抛出异常了。代码如下：

```java
package ThreadTrain;

import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

/**
 * 创建线程的第三种方法，使用Callable和Future创建线程
 * @author yanyu
 *
 */
public class demo3 
{
   public static void main(String[] args)
   {
      //创建Callable对象  1
      demo3 rt=new demo3();
      //先使用Lambda表达式创建Callable<Integer>对象  1
      //使用Futuretask来包装Callable对象
      FutureTask<Integer> task=new FutureTask<Integer>((Callable<Integer>)()->{
         //这里的方法相当于call()方法    1 call()方法可以有返回值  2 call()方法也可以声明抛出异常 
         int i=0;
         for(;i<100;i++)
         {
            //通过currentThread()来获得当前线程
            System.out.println(Thread.currentThread().getName()+" "+i);
         }
         //返回值
         return i;
      });
      //主线程
      for(int i=0;i<100;i++)
      {
         System.out.println(Thread.currentThread().getName()+" "+i);
         if(i==20)
         {
            //实质还是Callable对象创建线程的，执行者还是Thread
            new Thread(task,"有返回值的线程").start();
         }
      }
      //也可以获取异常
      try
      {
         //获取子线程的返回值get()方法
         System.out.println("子线程的返回值为："+task.get());
      }catch (Exception ex) {
         // TODO: handle exception
         ex.printStackTrace();
      }
   }
}
```

* 实现Runnbale接口和Callable接口获取当前线程都需要调用`Thread.currentThread()`方法。
* 运行Callable任务可以拿到一个Future对象，表示异步计算的结果。通过Future可以了解任务的执行情况和任务是否完成、也可以取消任务。

### 线程的状态流转图

![image-20210510164508318](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210510164508318.png)

​	一个线程刚开始创建就是new状态，只是简单的赋值，没有什么其它操作。调用线程的start方法后，线程进入运行态，如果此时的线程没有什么其它要求，就只是简单打印一句话，线程执行完毕后就进入了终止态。但是如果有特殊情况，它在语句中调用了object.wait()，这时它会进入等待状态，同时也进入了等待队列，就是waiting，另外一个线程如何调用了这个对象的notify()方法，最开始进入等待状态的线程会重新获取锁，进入运行态。如何这个时候等待锁的线程比较多，没有获取到的就进入阻塞状态，也就进入到了同步队列，一直等待获取到锁，然后进入运行态。还有一个状态就是超时等待状态，以线程的sleep方法为例，线程运行中执行这个方法，睡眠10秒，然后线程就进入了这个状态，10秒过完后，就再次回到运行态。但是这里不会放弃锁，会一直保持。

* 所有的状态都是先进入就绪态，再进入运行中。
* sleep方法不会放弃锁。
* 线程不能抛出任何异常，遇到异常，应该设置异常处理器。将异常的信息打印到日志中或者后续处理。
* Thread.yield()方法作用是：暂停当前正在执行的线程对象，并执行其他线程。线程状态为可运行态。
* Thread.join方法会让当前线程进入到等待状态，执行t，完毕后再执行当前线程。本质仍然是执行了 wait() 方法，而锁对象就是 Thread t 对象本身。
* park和unpark实现的原理则是使用线程内部的计数器。
* 用 jdk 的 Lock 接口中的 lock，如果获取不到锁，线程将挂起，状态则变为等待状态。
* jdk 中锁的实现，是基于 AQS 的，而 AQS 的底层，是用 park 和 unpark 来挂起和唤醒线程。

### 线程池相关知识点

​	创建一个线程池，现在里面没有任何运行的线程。提交一个任务，如果小于核心线程数，会一直创建新的核心线程。直到核心线程数满了，线程池还有一个任务队列，核心数满了后，会将提交的任务放到任务队列里面。当任务队列也满了后，会创建非核心线程数，当非核心线程数也满了后，就会走拒绝策略，将后面提交的任务全部拒绝。当非核心线程慢慢没有任务后，过一段时间就会对非核心线程进行销毁。核心线程则会一直运行中。

![image-20210607200224525](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210607200224525.png)

* 但是，敦促程序员使用更方便的Executors工厂方法Executors.newCachedThreadPool （无边界线程池，具有自动线程回收）， Executors.newFixedThreadPool （固定大小的线程池）和Executors.newSingleThreadExecutor （单个后台线程），这些方法可以预先配置设置。
* 当在方法execute(Runnable)提交新任务，并且正在运行的线程少于corePoolSize线程时，即使其他工作线程处于空闲状态，也会创建一个新线程来处理请求。 如果运行的线程数大于corePoolSize但小于maximumPoolSize，则仅在队列已满时才创建新线程。 通过将corePoolSize和maximumPoolSize设置为相同，可以创建固定大小的线程池。 通过将maximumPoolSize设置为一个本质上不受限制的值（例如Integer.MAX_VALUE ，可以允许池容纳任意数量的并发任务。 最典型地，核心和最大池大小仅在构造时设置，但也可以使用setCorePoolSize和setMaximumPoolSize动态更改。
* 使用ThreadFactory创建新线程。 如果没有另外指定，则使用Executors.defaultThreadFactory ，该线程创建的线程全部位于相同的ThreadGroup并且具有相同的NORM_PRIORITY优先级和非守护程序状态。 通过提供其他ThreadFactory，可以更改线程的名称，线程组，优先级，守护程序状态等。如果ThreadFactory在从newThread返回null返回要求时未能创建线程，执行器将继续执行，但可能无法执行执行任何任务。 线程应具有“ modifyThread” RuntimePermission 。 如果使用该池的工作线程或其他线程不具有此许可权，则服务可能会降级：配置更改可能不会及时生效，并且关闭池可能保持在可能终止但未完成的状态。
* 如果当前池中的线程数超过corePoolSize，则多余的线程将在空闲时间超过keepAliveTime时终止（请参见getKeepAliveTime(TimeUnit) ）。 当不积极使用池时，这提供了一种减少资源消耗的方法。 如果池稍后变得更活跃，则将构建新线程。 也可以使用setKeepAliveTime(long, TimeUnit)方法动态更改此参数。 使用Long.MAX_VALUE TimeUnit.NANOSECONDS的值Long.MAX_VALUE有效地使空闲线程永远不会在关闭之前终止。 默认情况下，仅当corePoolSize线程数多时，保持活动策略才适用。 但是，只要keepAliveTime值不为零，方法allowCoreThreadTimeOut(boolean)还可用于将此超时策略应用于核心线程。
* 有三种一般的排队策略：
  * 直接交接。 工作队列的一个很好的默认选择是SynchronousQueue ，它可以将任务移交给线程，而不必另外保留它们。 在这里，如果没有立即可用的线程来运行任务，则尝试将任务排队的尝试将失败，因此将构造一个新线程。 在处理可能具有内部依赖项的请求集时，此策略避免了锁定。 直接切换通常需要无限制的maximumPoolSizes以避免拒绝新提交的任务。 反过来，当平均而言，命令继续以比其可处理的速度更快到达时，这可能会带来无限线程增长的可能性。
  * 无限队列。 使用无界队列（例如，没有预定义容量的LinkedBlockingQueue ）将在所有corePoolSize线程繁忙时使新任务在队列中等待。 因此，将仅创建corePoolSize线程。 （因此，maximumPoolSize的值没有任何作用。）当每个任务完全独立于其他任务时，这可能是适当的，因此任务不会影响彼此的执行。 例如，在网页服务器中。 尽管这种排队方式对于消除短暂的请求突发很有用，但它承认当命令平均继续以比处理速度更快的速度到达时，无限制的工作队列增长是可能的。
  * 有界队列。 当与有限的maximumPoolSizes一起使用时，有界队列（例如ArrayBlockingQueue ）有助于防止资源耗尽，但调优和控制起来会更加困难。 队列大小和最大池大小可能会相互折衷：使用大队列和小池可以最大程度地减少CPU使用率，操作系统资源和上下文切换开销，但会导致人为地降低吞吐量。 如果任务频繁阻塞（例如，如果它们受I / O约束），则系统可能能够安排比您原先允许的线程更多的时间。 使用小队列通常需要更大的池大小，这会使CPU繁忙，但可能会遇到无法接受的调度开销，这也会降低吞吐量
*  在任一情况下， execute方法调用RejectedExecutionHandler.rejectedExecution(Runnable, ThreadPoolExecutor)其的方法RejectedExecutionHandler 。 提供了四个预定义的处理程序策略：
  * 在默认的ThreadPoolExecutor.AbortPolicy ，处理程序在拒绝时会抛出运行时RejectedExecutionException 。
  * 在ThreadPoolExecutor.CallerRunsPolicy ，调用execute自己的线程运行任务。 这提供了一种简单的反馈控制机制，该机制将减慢新任务的提交速度。
  * 在ThreadPoolExecutor.DiscardPolicy ，简单地删除了无法执行的任务。
  * 在ThreadPoolExecutor.DiscardOldestPolicy ，如果未关闭执行程序，则将丢弃工作队列开头的任务，然后重试执行（该操作可能再次失败，导致重复执行此操作）。

#### 参数讲解

* corePoolSize – 要保留在池中的线程数，即使它们处于空闲状态，除非设置了allowCoreThreadTimeOut

* maximumPoolSize – 池中允许的最大线程数

* keepAliveTime – 当线程数大于核心数时，这是多余空闲线程在终止前等待新任务的最长时间。
* unit – keepAliveTime参数的时间单位
* workQueue – 用于在执行任务之前保存任务的队列。 这个队列将只保存execute方法提交的Runnable任务。
* threadFactory – 执行程序创建新线程时使用的工厂
* handler – 执行被阻塞时使用的处理程序，因为达到了线程边界和队列容量。
* 线程池的状态：
  * RUNNING：接受新任务并处理排队的任务
  * SHUTDOWN：不接受新任务，但处理排队的任务
  * STOP：不接受新任务，不处理排队的任务，并中断进行中的任务
  * TIDYING：所有任务都已终止，workerCount为零，线程转换到TIDYING状态将运行Terminated（）挂钩方法。
  * TERMINATED：terminald（）已完成。

#### 常用方法

* execute(Runnable command)：在未来的某个时间执行给定的任务。
* shutdown()：启动有序关闭，其中执行先前提交的任务，但不会接受新任务。 如果已经关闭，调用没有额外的效果。
* shutdownNow() ：尝试停止所有正在执行的任务，停止等待任务的处理，并返回等待执行的任务列表。 
* submit():提交一个返回值的任务以供执行，并返回一个表示任务未决结果的 Future.

实现代码：

```java
package ThreadTrain;

import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * ThreadPoolTrain.java
 * Description: 线程池练习
 *
 * @author Peng Shiquan
 * @date 2021/6/7
 */
public class ThreadPoolTrain {
    public static void main(String[] args) {
        ThreadPoolExecutor executorService = (ThreadPoolExecutor) Executors.newFixedThreadPool(8);
        for (int i = 0; i < 100; i++) {
            TestTask testTask = new TestTask("线程" + i);
            executorService.execute(testTask);
            System.err.println("当前线程池核心线程数：" + executorService.getPoolSize());
            System.err.println("队列中等待的任务数为：" + executorService.getQueue().size());
            //返回已完成执行的大致任务总数。
            System.err.println("当前线程池完成的任务数为：" + executorService.getCompletedTaskCount());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class TestTask implements Runnable {
    String name;

    public TestTask(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.err.println("当前正在执行线程：" + name);
        try {
            Thread.sleep(4000);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.err.println("线程：" + name + "执行完毕");
    }
}
```

### 面试的问题

* 等待状态和阻塞状态的区别

​	线程的等待状态是主动的，自己调用wait方法将线程切换为等待状态，并且这个时候也会占用cpu，阻塞状态是被动的，获取不到锁就变成阻塞状态，这个时候会放弃cpu的执行权，等待某个时间将其唤醒。

* 重复调用start方法会怎么样？

​	会抛出异常，就在start方法的第一行，判断状态如果不是new的话，会直接抛出一个异常。

* 线程池的创建方式有几种

​	创建线程池第一种就是自定义创建，指定参数，创建线程池。还有一种就是通过Executors工具类创建指定好的线程池。一般三种类型的，线程数没有限制的、线程数固定大小的、线程数为一个的。看自己业务的需求，但是阿里的编码规范里面说不要使用工具类创建，是为了防止内存溢出。不知道我们公司线程池的使用是那种方式？

* 线程池的拒绝策略有哪些？

​	我了解的拒绝策略有一下啊几种：1.队列满了后直接抛出异常。2.无法执行的任务丢弃掉。3.使用调用者线程执行任务，这样能够减慢任务的提交速度。4.将队列头部的任务丢弃掉。

* 
