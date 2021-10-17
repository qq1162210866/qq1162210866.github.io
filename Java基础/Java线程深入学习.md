## Java线程深入学习

​	上一篇博客讲了一下并发，但是没有达到自己的效果，所以这篇博客想从上一篇博客中的线程讲起，打算写一篇有着自己风格和能够让自己满意的博客。也是想深入了解一下线程的工作原理。

​	线程是程序中的执行线程。 Java虚拟机允许应用程序具有多个并发运行的执行线程。Java创建线程有两种方式（Java文档中这样说的，网上说应该还有一种,总共三种）一种是继承`Thread`类，一种是实现`Runnable`接口。下面先就第一种创建方法来说一下线程具体的流程。现在上代码。

```java
package ThreadTrain;

/*
 * 第一种创建线程的方法，通过继承Thread类创建线程类
 */
public class demo1 extends Thread {
    private int i;//计数器

    //run方法就是线程执行体
    @Override
    public void run() {
        for (; i < 100; i++) {
            /*
             * 当线程类继承Thread类时，直接使用this可以获得当前线程
             */
            //获取当前线程的名字
            System.out.println(getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            /*
             * 运行的时候发现main的i不一定会到20，线程的启动是计算机调度的，具有一定的随机性
             * 两个子线程的i不是共享的，不会连续打印
             */
            //currentThread()总是返回当前正在执行的线程对象
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 20) {
                //通过start()方法启动第一个线程
                new demo1().start();
                //启动第二个线程
                new demo1().start();
            }
        }
    }
}
```

​	最后运行的截图如下：

![image-20210112173640626](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210112173640626.png)	

​	基础的知识就不再详细介绍了，继承`Thread`类，重写`run()`方法，通过调用该类对象的`start()`方法（不是`run(`)方法）来启动线程。剩下就没有什么需要讲解的了，但是这篇博客是深入了解线程，所以会继续深挖一下，了解背后的原理。下面就开始阅读线程类的源码。

​	上面的过程涉及了几个地方，一个是创建一个对象，一个就是调用了`start()`方法。下面就一步一步来，先是new了一个对象，看到源码中是调用了初始化方法，只是初始化了线程的相关设置，和我们的关系好像不大。（其中涉及的感觉都是JVM相关的东西）

![image-20210112175323837](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210112175323837.png)

​	下面就是`start()`方法了，先看该方法的介绍，显示该方法会使线程开始执行，Java虚拟机将会调用此线程的run方法，结果是产生两个线程（调用start方法的线程和执行run方法的线程），线程一旦完成就可能不会重启。再来看代码，发现代码不是很复杂：将该线程放入线程组，再调用`start0()`方法，然后到这里就断了，这里就调用了底层C的代码，再看下去就不是很容易懂了。好像到这里就暂时断了。

​	回过头来，就不难发现为什么线程的名称是`Thread-0`,因为初始化时没有传参所以系统给默认的了。同时也能解释代码中注释的疑问：运行次数多不难发现，不一定运行到20就开始打印子线程？和主线程的打印和子线程是随机不确定间隔打印的。（因为线程的启动时随机的，主线程和子线程是两个独立的线程）

​	下面就来看看创建线程的第二种方式。实现`Runnable`接口，直接上代码。

```java
package ThreadTrain;
/*
 * 创建线程的第二种方法
 * 实现Runnable接口创建线程
 */
public class demo2 implements Runnable 
{
   private int i;//初始值为0
   //线程执行体
   @Override
   public void run() 
   {
      for(;i<100;i++)
      {
         //当实现Runnable接口时，只能使用Thread.currentThread().getName()获得当前线程
         System.out.println(Thread.currentThread().getName()+" "+i);
      }
   }
   public static void main(String[] args)
   {
      for(int i=0;i<100;i++)
      {
         System.out.println(Thread.currentThread().getName()+" "+i);
         if(i==20)
         {
            demo2 dt=new demo2();
            /*
             * 最终的执行者还是Thread
             * 发现新线程1和新线程2的i值是连续的，因为Thread共享了同一个target(就是Runnable对象)
             */
            //通过new Thread(target,name)来创建新线程
            new Thread(dt,"新线程1").start();
            new Thread(dt,"新线程2").start();
         }
      }
   }
}
```

​	代码也是比较简单，但是创建线程这里不太一样，通过创建实现了Runnable接口的对象，但是实际还是调用Thread线程类的`start()`方法。看源码发现虽然调用的是不同的构造函数，但是最后调用的都是同一个初始化函数，而且传入的Runnable好像也没有用到，只是赋值了一下，就不再处理，那使用接口和不使用接口有什么区别呢？继续阅读源码。

​	如果实现接口，代码中并没有重写Thread类的`run()`方法，只是重写了Runnable的`run()`方法，回到Thread类的run方法，发现有这样的代码。

![image-20210113174447700](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210113174447700.png)

​	这样就明白了，Thread是先调用Runnable的run方法，也就是说线程的创建还是在Thread中，Runnable只是将自己作为接口放到了Thread的上面。回到最上面也可以看到Thread是实现了Runnable接口的。Runnable接口也比较简单，就只有一个run方法。

​	回到刚才的问题，使用接口和继承Thread有什么区别，其实就是代码为什么这样设计的问题。看了一下Runnable的文档，如下：

![image-20210113175122171](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210113175122171.png)

​		说白了，就是更加的灵活，Thread有一千行代码，其中很多代码可能是大家用不到的，这样也体现了一种思想，去除无用（自己想的）。让代码中尽量少一些无用代码。

​	代码其实比较简单，但是其中的一些设计思想需要消化一下。能力有限，只能考虑到这些，欢迎大家补充。

​	最后来看一下网上说的第三种创建线程的方式，使用Callable和Future创建线程。代码如下：

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

​	按照上面的方式继续阅读这段代码，不难发现`FutureTask<V>`其实是实现了Runnable的子类，这样的话和第二种方法就没有什么区别，这里也不再继续跟进。所以最开始文档中说的对，实现线程的方式只有两种。

​	这篇博客写到这里算是一个完结，因为目前能力有限，可能一些理解不太合适。也想了一下这种情况，个人来说很难学习一个知识点就直接学习到深处，很多东西都是循序渐进的，这样也就能解释之前的博客水平不咋地。一个高度就会有一个理解，不同高度的博客很难相比较。所以后续肯定也会回头看这篇博客。

​	博客是分两天写完的，中间间断了一下，但是最后的结果感觉还不是很满意，中间的语句和语序都有一些问题，也留一个结论。博客最好一天写完，可以后续完善，但是一定要一天定下文章的结构和大体框架。算是写作的一个小提示吧。

​	就这样吧，结束。