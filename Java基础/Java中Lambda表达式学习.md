 ## Java中Lambda表达式学习

​	上一篇博客了解了线程的相关学习，学习的过程中看到一个注解`@FunctionalInterface`，本次的博客就来学习一下函数式接口。函数式接口可以用Lambda来创建，所以函数式接口的学习应该在Lambda表达式前面。

​	先来看一下函数式接口的文档定义：

![image-20210218093905631](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210218093905631.png)

可以发现几个重要的点：

* 函数式接口只有一个抽象方法。
* 默认方法和覆盖方法都不计入抽象方法数目。
* 可以用Lambda表达式、方法引用、构造函数引用来创建函数式接口实例。
* 注解并不一定是必须的，只要符合要求，编译器会自动将接口视为函数式接口。

​	下面看一个jdk中自带的函数式接口的实现，`Comparator`接口。发现里面有很多方法的实现，但是也使用了`@FunctionalInterface`注解，仔细再看，可以看到只有两个抽象方法，其他都是静态方法或者默认方法，都有实现。再看两个抽象方法，一个是继承了Object类的`equals`方法，所以最后这个接口只有一个抽象方法，符合函数式接口的定义。

​	下面就来看一下如何使用。代码如下：

```java
 /**
     * Description: 使用lambda表达式作为转换为函数式接口传递到Arrays.sort中第二个参数
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2019-07-18
     */
    public void arraySort() {
        String[] arrays = {"1237", "123"};
        Arrays.sort(arrays, (first, second) -> second.length() - first.length());
        System.err.println(arrays[0]);
    }
```

​	方法描述已经比较详细的介绍了这个方法的作用，因为`sort(T[] a, Comparator<? super T> c)`方法不是本篇博客的重点，所以不再叙述太多，简单理解为将数组中的元素按照给定的方法比较排序即可。这里第二个参数就是函数式接口，可以看到这里的传值为`(first, second) -> second.length() - first.length()`,这个就是典型的Lambda表达式的应用，函数式接口的实现内容就是：返回用第二个参数的长度减去第一个参数的长度得到的值。但是我们还是有很多疑问的。下面一一去寻找答案。

* `(first, second) -> second.length() - first.length()`是什么意思，为什么这样写？换个方式`(String first, String second) -> {return second.length() - first.length();}`这样是不是就可以看出大概了，知道这个是`Comparator`中接口方法`int compare(T o1, T o2);`的实现。
* 为什么不用声明参数类型，代码是如何知道我传入的类型？这个需要看一下方法`public static <T> void sort(T[] a, Comparator<? super T> c)`，可以看到在调用soft方法时就已经声明了参数的类型，所以编译器就知道了这里是否正确，也是可以省略参数类型的原因。

​	Lambda表达式的特征如下：

![image-20210218113001221](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210218113001221.png)

​	下面开始编写自己的一个函数式接口的demo。代码如下：

```java
package LambdaTrain;

public class demo1 
{
	/*
	 * 测试使用的代码，分别要传入各自的参数
	 */
	public void eat(Eatable e)
	{
		System.out.println(e);
		e.taste();
	}
	public void drive(Flyable f)
	{
		System.out.println("我正在驾驶："+f);
		f.fly("【碧空如洗的日子】");
	}
	public void test(Addable add)
	{
		System.out.println("5与3的和为："+add.add(5, 3));
	}
	/*
	 * 主方法，其中的方法参数都是使用lambda表达式
	 */
	public static void main(String[] args)
	{
		demo1 demo1=new demo1();
		//lambda表达式的代码只只有一句，可以省略“{}”
		//这里的lambda表达式实际上被当成任意的类型，具体的类型取决于运行环境的需要
		demo1.eat(()->System.out.println("苹果的味道不错"));
		//方法的形参只有一个，可以省略“()”  weather是接口方法的形参
		demo1.drive(weather->{
			System.out.println("今天的天气是："+weather);
			System.out.println("直升机飞行平稳");
		});
		//代码只有一句，可以省略“{}”,同时也可以省略return关键字 a+b是返回值
		demo1.test(Integer::sum);
		
	}

}
/*
 * 测试使用的接口,只有一个抽象方法的接口（函数式接口）
 */
@FunctionalInterface
interface Eatable
{
	void taste();
}
@FunctionalInterface
interface Flyable
{
	void fly(String weather);
}
@FunctionalInterface
interface Addable
{
	int add(int a,int b);
}

```

​	现在看这些代码是不是很容易就明白了。这里也就不再将代码进行详细解析。

​	函数式接口和其如何使用都已经了解了，下面就开始了解函数式接口有哪些使用方式，从最开始的文档中可以了解到，有三种方式去创建函数式接口实例。lambda表达式，方法引用或构造函数引用。Lambda表达式刚才已经看到并且使用到了，下面详细说一下方法引用和构造函数引用。

* 方法引用。方法引用分为实例方法引用、静态方法引用和特定类型的方法引用。语法分别为：

  * 实例方法引用：`new instMethod()::method`
  * 静态方法引用：`类名::staticMethod`
  * 特定类型的方法引用：`类名::instMethod`（个人认为不常用）

  举例代码如下：

  ```java
  package LambdaTrain;
  
  import java.util.function.BiConsumer;
  import java.util.function.Consumer;
  import java.util.function.Supplier;
  
  /**
   * LambdaTrain6.java
   * Description: 博客中使用的练习demo，将方法引用汇集在一起
   *
   * @author Peng Shiquan
   * @date 2021/2/18
   */
  public class LambdaTrain6 {
  
      /**
       * Description: 第一个<T> 声明范型的类型
       *
       * @param s
       * @return void
       * @Author Peng Shiquan
       * @Date 2021-02-18
       */
      static <T> void hello(T s) {
          System.err.println("这是一个静态方法引用，有参数，但是没有返回值：" + s);
      }
  
      String put() {
          return "这是一个普通方法引用，没有参数，但是返回值";
      }
  
      void fun(String s) {
          System.out.println("这是特定类型的方法引用" + s);
      }
  
      public static void main(String[] args) {
          //静态方法引用
          Consumer<String> consumer = LambdaTrain6::hello;
          consumer.accept("hello");
          //普通方法引用
          Supplier<String> supplier = new LambdaTrain6()::put;
          System.err.println(supplier.get());
          /**
           * 1.抽象方法的第一个参数类型刚好是实例方法的类型（函数式接口的抽象方法必须要有输入参数）。2.抽象方法剩余
           * 的参数恰好可以当做实例方法的参数。如果函数式接口的实现能由上面说的实例方法调用来实现的话，
           * 那么就可以使用对象方法的引用(两个条件都要满足)
           */
          BiConsumer<LambdaTrain6, String> biConsumer = LambdaTrain6::fun;
          biConsumer.accept(new LambdaTrain6(), "hello");
      }
  }
  
  ```

 * 构造方法引用 ，构造方法引用的语法就只有一种：`类名::new`

   举例代码如下：

   ```java
   package LambdaTrain;
   
   import java.util.function.Consumer;
   import java.util.function.Function;
   import java.util.function.Supplier;
   
   /**
    * LambdaTrain5.java
    * Description: lambda表达式练习，练习方法引用的构造方法引用
    *
    * @author Peng Shiquan
    * @date 2019-08-13
    */
   public class LambdaTrain5 {
       /**
        * 如果函数式接口的实现恰好可以通过调用一个类的构造方法来实现，那么就可以使用构造方法引用
        * 语法
        * 类名::new
        */
   
       /**
        * Description: 主方法
        *
        * @param args
        * @return void
        * @Author: Peng Shiquan
        * @Date: 2019-08-13
        */
       public static void main(String[] args) {
           Supplier<Person> supplier = () -> new Person();
           Supplier<Person> supplier1 = Person::new;
           supplier.get();
           supplier1.get();
   
           Consumer<Integer> consumer = (i) -> new Student(i);
           Consumer<Integer> consumer1 = Student::new;
           consumer.accept(1);
           consumer1.accept(2);
   
           /**
            * 有参构造器
            */
           Function<String, Integer> function = s -> s.length();
           Function<String, Integer> function1 = String::length;
   
           System.out.println("this method have one parameter" + function.apply("hello"));
           System.out.println("this method have one parameter" + function1.apply("test"));
   
           Function<String, Student> function2 = Student::new;
           function2.apply("hahahah");
   
   
       }
   
   }
   
   class Person {
       /**
        * Description: 重写构造方法
        *
        * @param
        * @return
        * @Author: Peng Shiquan
        * @Date: 2019-08-13
        */
       public Person() {
           System.out.println("this is Person's construction method");
       }
   }
   
   class Student {
       public Student() {
           System.out.println("this method have no parameter ");
       }
   
       public Student(int i) {
           System.out.println("this method have one parameter" + i);
       }
   
       public Student(String s) {
           System.out.println("this method have one parameter" + s);
       }
   }
   ```

​	学习了函数式接口的使用方式和练习了一些基本的demo，现在回头看Lambda表达式是不是就更加清楚其意思了。但是什么时候或者什么地方该用Lambda呢？这个问题我现在也没有很明白，后续会慢慢补充。`java.util.function`包下都是jdk自带的函数式接口，阅读完这些接口的含义可能会对函数式接口有了一个更加清晰的认识，说到底，Lambda表达式和函数式接口的存在还是为了让写代码更加清晰和简洁。万变不离其意。要在开发中你感觉可以使用Lambda的地方使用它，使用的多了，了解也就更加的深入了。

​	就这样吧，结束。

